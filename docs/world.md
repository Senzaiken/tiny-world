# World

## Planet mesh

- An icosphere (`DETAIL` 127) with merged vertices: 163,842 vertices and 327,680 triangles, about 1.4° between neighbouring vertices.
- Each vertex stores a unit direction and a height. Its position is `direction × (1 + height)`.
- Sea level is height 0. Water is a separate translucent sphere at radius 1.
- Flat shading gives the low-poly look. Vertex colours carry the biomes.
- Vertex normals are kept on the CPU only, for slope colouring. They're recomputed locally around each brush stroke.
- Weather recolouring is spread over about 6 frames, so snow and rain never cause a hitch.

## Generation

Generation is seeded, so a seed always rebuilds the same world.

1. **Domain warp.** Low-frequency noise bends the sample point, which makes coastlines wander.
2. **Continents.** 6-octave fBm, offset by −0.04, gives about 44% land.
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
- Orbit controls: drag to spin, pinch to zoom (1.55 to 14), no panning. The starting distance is fitted to the screen's narrow side.
