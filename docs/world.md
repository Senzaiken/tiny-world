# World

## Planet mesh

- An icosphere built in-house: each of the icosahedron's 20 faces is a triangular grid with 256 segments per edge (`S`). That's 655,362 vertices and 1,310,720 triangles, about 0.7° between neighbours. It uses the same lattice as `THREE.IcosahedronGeometry(1, 255)`, which is why saves record `detail: 255`.
- Each vertex stores a unit direction and a height. Its position is `direction × (1 + height)`.
- Sea level is height 0. Water is a separate translucent sphere at radius 1.
- Smooth shading, with normals averaged from the surrounding triangles. Vertex colours carry the biomes.
- Heights, normals and colours live in global arrays. The chunks below copy from them.

## Chunks

- Every face is cut into blocks of 64×64 grid cells (`CHUNK`): 10 blocks per face, 200 chunks in all, each with at most 4,225 vertices and 8,192 triangles. Each chunk is its own mesh with its own raycast tree (three-mesh-bvh).
- **Edits** recompute normals and colours only for the changed vertices and their neighbours. Only the chunks containing them are re-uploaded and refit.
- **Weather recolouring** walks 28 chunks per frame (about 7 frames for the whole world), so snow and rain never cause a hitch.
- **Culling** happens every frame, and a chunk is drawn only if it passes both checks:
  - **Horizon:** it can be seen over the sea-level horizon from where the camera is. This allows for the chunk's angular size and its tallest point, since a peak can show over the horizon.
  - **View:** its bounding sphere is inside the camera's view.
- At the default zoom about two thirds of the chunks are drawn. Close up it's usually 25 to 30. The Sculpt panel shows the live count.

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
  - Speed is 4.4 rad per screen width, scaled down when zoomed in.
  - Letting go keeps the spin at the release speed: the rate is smoothed over the last few moves, and holding still before letting go means no spin. It then slows by 42%/s proportional plus 0.12 rad/s² constant drag, capped at 30 rad/s. A hard swipe glides for several seconds.
  - Touching the planet again catches it.
- Pinch or the mouse wheel zooms (distance 1.55 to 14, eased). The starting distance is fitted to the screen's narrow side.
- A loading screen shows while the world is generated (about a second on a phone).

## Fling

Swiping fast makes the planet squash and stretch like a cartoon, and throws the clouds outward. Both spring back with a wobble.

- The spin rate and axis come from the globe's own angular velocity: the drag rate while a finger is down, the glide afterwards. The axis follows quickly and holds while the spin dies down.
- Slow and medium spins do nothing. A careful drag peaks at about 4 rad/s, while a real swipe is 15 to 30.
- **Squash spring** (stiffness 170, damping 9): it starts above 7 rad/s and reaches 0.12 at about 16 rad/s. The planet shrinks by that fraction along the spin axis and bulges around it, keeping its volume. This is a matrix on the `squash` group, which sits outside the spinning globe so the bulge stays aligned with the axis while the planet turns inside it.
- **Cloud spring** (stiffness 45, damping 4.5): it starts above 6 rad/s and reaches 0.16. Clouds rise by that fraction of their altitude, and their wisps grow by 80% of it. Rain and snow follow the clouds.
- The atmosphere glow isn't squashed.
