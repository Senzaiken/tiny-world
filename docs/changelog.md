# Changelog

## 2026-11-01: Higher-resolution planet

- The planet mesh is 4× finer: `DETAIL` went from 63 to 127, which means 163,842 vertices. Coastlines, ridges and sculpting are much smoother.
- Sculpting stays fast at the new size:
  - Picking uses a bounding-volume tree (three-mesh-bvh).
  - Normals and colours update only around the brush.
  - Weather recolouring is spread across frames.
- Existing sculpted worlds are upsampled onto the new mesh when they load, so nothing is lost.
- Undo now keeps 15 strokes instead of 20.

## 2026-11-01: First world

- Procedural planet: warped continents, inland mountain ranges, biome colours, translucent sea, atmosphere glow with a day/night tint, stars, and a slow spin.
- Weather buttons: Clear, Cloudy, Rain, Storm, Snow and Fog. Orbiting instanced clouds; rain and snow fall from them; lightning in storms; a fog layer; snow settles and melts, and rain darkens the ground.
- Sculpt mode: Raise, Lower, Smooth and Flatten brushes, with size, strength and a 20-step undo.
- Saves the seed, weather and sculpted heights to `localStorage`.
- Home-screen icon, web manifest, and a new-version banner driven by `version.json`.
