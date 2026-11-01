# World

## Planet mesh

- An icosphere built in-house: each of the icosahedron's 20 faces is a triangular grid with 256 segments per edge (`S`). That's 655,362 vertices and 1,310,720 triangles, about 0.7° between neighbours. It uses the same lattice as `THREE.IcosahedronGeometry(1, 255)`, which is why saves record `detail: 255`.
- Each vertex stores a unit direction and a height. Its position is `direction × (1 + height)`.
- Sea level is height 0. Water is a separate translucent sphere at radius 1 (see Water below).
- Smooth shading, with normals averaged from the surrounding triangles. Vertex colours carry the biomes.
- **Surface detail** is drawn per pixel by the terrain shader: five octaves of 3D value noise fixed to the planet, at 90 up to about 2,800 cycles per planet radius.
  - It varies the colour and bumps the lighting. Rock gets strong grain, grass gets speckle, and bright sand and snow stay smooth. The kind of ground is read from the vertex colour's saturation and brightness.
  - Each octave fades in only once it's bigger than about a pixel, so detail sharpens as you zoom in and costs nothing far away.

## Water

The sea is a standard PBR sphere (roughness 0.1, opacity 0.8) with waves added in its shaders:

- Eight travelling sine waves, fixed to the planet's surface, at frequencies 32 to 350 per radius.
- They tilt the normal, which gives rolling highlights and a glittering sun path. Strength is 1.8 + 0.9 × wind.
- They also lift the surface by up to about 0.005 (scaled 0.6 + 0.4 × wind), so shorelines lap.
- Each wave fades out once it's finer than a pixel, so distant water doesn't alias into sparkle grids.
- A flat sky-blue environment map gives reflection at grazing angles (intensity 0.55, less in grey weather).
- The water turns slate in storms.
- Heights, normals and colours live in global arrays. The chunks below copy from them.

## Chunks

- Every face is cut into blocks of 64×64 grid cells (`CHUNK`): 10 blocks per face, 200 chunks in all, each with at most 4,225 vertices and 8,192 triangles. Each chunk is its own mesh with its own raycast tree (three-mesh-bvh).
- **Edits** recompute normals and colours only for the changed vertices and their neighbours. Only the chunks containing them are re-uploaded and refit.
- **Weather recolouring** walks 28 chunks per frame (about 7 frames for the whole world), so snow and rain never cause a hitch.
- **Culling** happens every frame, and a chunk is drawn only if it passes both checks:
  - **Horizon:** it can be seen over the sea-level horizon from where the camera is. This allows for the chunk's angular size and its tallest point, since a peak can show over the horizon.
  - **View:** its bounding sphere is inside the camera's view.
- At the default zoom about two thirds of the chunks are drawn. Close up it's usually 25 to 30. The Sculpt panel shows the live count, plus how many chunks are drawn at extra detail.

## Adaptive detail

Each chunk can be redrawn at level 1 (2× per edge, up to 33k triangles) or level 2 (4× per edge, up to 131k triangles). The base mesh stays the source of truth: sculpting, picking and saving all work on it.

- **Interest** caps how far a chunk can be refined. It's recomputed after generation, after each stroke and after undo.
  - Level 2: the chunk's highest point is over 0.06, *or* its land's average tilt (1−cos) is over 0.25, *or* over 12% of its vertices are within 0.005 of sea level.
  - Level 1: thresholds of 0.015, 0.1 and 3%.
  - Open sea and flat plains stay at level 0. Seabed slopes don't count.
- **Screen size** sets what a visible chunk needs: aim for base cells of 3 px or less once refined. A cell over 3.6 px asks for level 1, and over 7.8 px for level 2. At the default zoom nothing is refined; close up the nearest chunks reach level 2.
- **Budget:** at most 40 chunks above level 0, of which 10 at level 2, nearest chunks first.
- **Building** runs in the background, 7 ms per frame, and pauses during a stroke. Desktop timings are about 115 ms for a level-2 chunk and 10 to 35 ms for level 1. Until a build is ready the chunk shows its best finished level. A sculpted chunk drops back to the base mesh until it's rebuilt.
- **How fine points are placed:**
  - Points between base vertices are curved with Phong tessellation (α 0.75) from the base normals.
  - They then get extra relief: up to 0.0045 on mountains (from height 0.025 to 0.08) and 0.0016 around the coast, from 3 octaves of noise at 90 per radius. The relief fades out within 1.5 base cells of the chunk edge.
- **Chunk edges** stay on the straight base edges, with base normals, so chunks at different levels meet without cracks or lighting seams.
- **Memory:** fine meshes nobody has wanted for 8 seconds, or that went stale, are freed.

## Generation

Generation is seeded, so a seed always rebuilds the same world.

1. **Domain warp.** Low-frequency noise bends the sample point, which makes coastlines wander.
2. **Continents.** 6-octave fBm, offset by −0.04, gives roughly 40 to 50% land.
   - Ocean: `height = continent × 0.2`, floored at −0.09.
   - Land: `0.004 + continent × 0.05`.
3. **Mountains.** Ridged noise is added only inland, and only inside broad "range" regions picked by a second low-frequency noise. It adds up to about 0.12.
4. **Detail.** A small high-frequency ripple (±0.0015).

## Colours

Colours come from height, slope and latitude:

- Seabed: deep blue to turquoise in the shallows.
- Beach: sand.
- Lowland: grass, with two shades mixed by a variation noise.
- Above about 0.02: forest.
- Above about 0.045, or on steep slopes: rock.
- Snow sits above the snowline (0.088). The snowline drops toward the poles, and further as snow accumulates (see [Weather](weather.md)).
- Wet ground (rain) darkens land by up to 24%.

## Atmosphere

A single front-facing shell at radius 1.24 with a custom shader. For each pixel it finds how closely the view ray passes the planet's centre:

- Outside the planet's disk, the result is a glow ring that fades out toward the shell's edge.
- Over the disk, it adds a haze that thickens toward the limb.
- Brightness follows the sun side, with a warm tint along the day/night line.
- Colour and strength change with the weather.

## Scene and camera

- The planet is tilted 0.38 rad and spins slowly (0.035 rad/s). Spinning pauses in Sculpt mode.
- One warm directional sun plus a hemisphere fill light.
- 1,800 background stars.
- The camera stays put. Dragging spins the world itself (`globe`, which holds the planet, clouds, rain and lightning) around the axis perpendicular to the drag. That works on any axis, over the poles too. The sun and atmosphere stay fixed, so spinning sweeps day and night across the planet.
  - Speed is 2.2 rad per screen width, scaled down when zoomed in.
  - Letting go keeps the spin at the release speed: the rate is smoothed over the last few moves, and holding still before letting go means no spin. It then slows by 42%/s proportional plus 0.12 rad/s² constant drag, capped at 18 rad/s. A hard swipe glides for several seconds.
  - Touching the planet again catches it.
- Pinch or the mouse wheel zooms (distance 1.55 to 14, eased). The starting distance is fitted to the screen's narrow side.
- A loading screen shows while the world is generated (about a second on a phone).

## Fling

Swiping fast makes the planet squash and stretch like a cartoon, and throws the clouds outward. Both spring back with a wobble.

- The spin rate and axis come from the globe's own angular velocity: the drag rate while a finger is down, the glide afterwards. The axis follows quickly and holds while the spin dies down.
- Slow and medium spins do nothing. A careful drag peaks at about 2 rad/s, while a real swipe is 8 to 15.
- **Squash spring** (stiffness 170, damping 9): it starts above 4 rad/s and reaches 0.12 at 9 rad/s. The planet shrinks by that fraction along the spin axis and bulges around it, keeping its volume. This is a matrix on the `squash` group, which sits outside the spinning globe so the bulge stays aligned with the axis while the planet turns inside it.
- **Cloud spring** (stiffness 45, damping 4.5): it starts above 3.5 rad/s and reaches 0.16 at 8.5 rad/s. Clouds rise by that fraction of their altitude, and their wisps grow by 80% of it. Rain and snow follow the clouds.
- The atmosphere squashes with the planet. Its mesh sits in the `squash` group, and its shader undoes the squash (`uInv`) so the glow maths still runs on a unit sphere.
