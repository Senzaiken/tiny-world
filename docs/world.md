# World

## Planet mesh

- An icosphere built in-house: each of the icosahedron's 20 faces is a triangular grid with 256 segments per edge (`S`). That's 655,362 vertices and 1,310,720 triangles, about 0.7° between neighbours. It uses the same lattice as `THREE.IcosahedronGeometry(1, 255)`, which is why saves record `detail: 255`.
- Each vertex stores a unit direction and a height. Its position is `direction × (1 + height)`.
- Sea level is height 0 (it rises in rain; see the weather page). The water is drawn on the terrain's own chunks (see Water below).
- Smooth shading, with normals averaged from the surrounding triangles. Vertex colours carry the biomes.
- **Surface detail** is drawn per pixel by the terrain shader: five octaves of 3D value noise fixed to the planet, at 90 up to about 2,800 cycles per planet radius.
  - It varies the colour and bumps the lighting. Rock gets strong grain, grass gets speckle, and bright sand and snow stay smooth. The kind of ground is read from the vertex colour's saturation and brightness.
  - Each octave fades in only once it's bigger than about a pixel, so detail sharpens as you zoom in and costs nothing far away.

## Water

The sea has no mesh of its own. Every chunk has a second mesh that shares the chunk's geometry and draws it with the water material: the vertex shader moves each terrain vertex out to the sea surface, and the original height gives the depth there. Fine-detail chunks get a water mesh on their fine geometry too. Chunks whose lowest point is above 0.03 skip water.

- **Depth:** turquoise in the first 0.014 of depth, deep blue below. Opacity goes from 0.3 in the shallows to 0.88 in deep water, and fades to zero right at the waterline. Because the water and the land use the same triangles, the fade follows the true shoreline exactly.
- **Surf:** a thin white lip at the waterline, plus foam bands parallel to the shore that roll inward within 0.004 of depth, broken up by noise. Bands fade out when they'd be finer than a pixel. Foam is rough, not shiny. Lakes get no surf bands and a fainter lip.
- **Ripples** tilt the normal by *analytic* gradients. Screen-space derivatives looked grainy, so they aren't used. There are two parts:
  - three octaves of drifting value noise, from 36 per radius;
  - on the ocean, three wind-wave trains (110, 176 and 282 per radius) travelling downwind, spread ±25°, scaled by the wind's strength and broken up by slow noise.
  - Every layer fades out before it shrinks to a few pixels, so the sea stays smooth. Overall strength is 1.8 + 0.9 × wind.
- **Swell and tides** move the surface itself:
  - four slow travelling waves in different directions (frequencies 22 to 63, amplitudes 0.0009 to 0.0004, weaker on lakes). They're sines rather than noise so the ship can compute the same surface in JS (`SWELL`, `seaOffset`);
  - on the ocean, a wind swell of long crests rolling downwind (about 0.0016 × wind strength);
  - a tide of two bulges circling the planet every 31 seconds (0.0011, highest at the equator).
  - Together these make shorelines lap and slowly rise and fall.
- **Lakes and oceans:** see below.
- A flat sky-blue environment map gives reflection at grazing angles (intensity 0.55, less in grey weather). The water turns slate in storms.
- **Wet ground:** the terrain shader darkens a band of about 0.002 either side of the current sea level, by up to 20%.

## Lakes and oceans

- A flood fill over every vertex below sea level finds each separate body of water.
- Bodies with at least 1% of the planet's vertices are ocean. Smaller ones are lakes.
- Land vertices touching a lake count as lake too, so the shoreline triangles agree.
- The label is a per-vertex `aLake` attribute on every chunk mesh, fine ones included.
- It's recomputed after generation, and after any stroke or undo that moved ground across sea level. Fine meshes in changed chunks rebuild.
- Lakes are darker and greener, calm, with no surf bands, no wind swell and no tides.

## Winds

Prevailing winds circle the planet's spin axis in Earth-like belts. `u` is eastward and `v` is northward; the same formula runs in JS (`zonalWind`) and in the water shader (`windAt`):

- `u = −cos(3·lat)`: easterly trades below 30°, westerlies from 30° to 60°, polar easterlies above.
- `v = −0.35·sin(6·lat)`: the trades blow toward the equator, the westerlies poleward, the polar winds toward the equator.
- Where the belts meet (about 30° and 60°) the wind is calm, and so is the sea.
- Ocean wind waves and swell travel downwind, scaled by the wind's strength.
- Clouds drift with `u` (times their own speed and the weather's wind): westward in the tropics, eastward in the mid-latitudes. Clouds, rain, snow, lightning and cirrus live in the tilted planet frame so their latitudes match the belts.
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
   - Land: `0.004 × smoothstep(0, 0.035, continent) + continent × 0.05`. That's a gentle ramp up from the shore, with no step at sea level, which used to make stair-stepped coastlines.
3. **Mountains.** Ridged noise is added only inland, and only inside broad "range" regions picked by a second low-frequency noise. The crests are rounded (`1 − √(n² + 0.012)` instead of `1 − |n|`), and it adds up to about 0.1.
4. **Detail.** A small high-frequency ripple (±0.0015).
5. **Softening.** Two relaxation passes pull ground above 0.012 halfway toward its neighbours' average, fully from 0.03 up, so peaks and ridges round off. The result is the base that saves are compared against.

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
- One warm directional sun, a hemisphere fill light, and a soft bluish ambient light (0.32) so slopes facing away from the sun never go black.
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
