# Changelog

## 2026-11-01: Lakes, trade winds, smoother sea and mountains

- **Lakes vs oceans:** separate bodies of water are found by flood fill. Small enclosed ones are lakes: calm, darker green-blue, with no surf, tides or wind swell.
- **Trade winds:**
  - Earth-like wind belts: easterly trades, westerlies and polar easterlies, with calm belts between.
  - Ocean waves and swell travel downwind.
  - Clouds drift westward in the tropics and eastward in the mid-latitudes.
  - Two tidal bulges circle the planet, so coasts slowly rise and fall.
- **Smoother water:** ripples now use analytic gradients and only broad layers, so the grain and sparkle specks are gone.
- **Softer mountains:**
  - rounded ridge crests and slightly lower peaks;
  - a smoothing pass on high ground (this changes the generated terrain slightly; sculpting is kept);
  - gentler rock bump;
  - a soft fill light so shadowed slopes are never pitch black.
- **No dark rim at coasts:** the steep drop-off just offshore no longer paints the shore as rock.

## 2026-11-01: Smooth coasts and a real shoreline

- **Smooth coastlines:** the generator no longer puts a small cliff at sea level, which made coasts snap to the mesh grid and show a dark stair-step edge. Land now ramps gently up from the water. Sculpted worlds keep their sculpting, though shorelines shift slightly.
- **The sea is drawn on the terrain chunks** and knows its depth everywhere:
  - clear turquoise shallows,
  - a soft fade at the waterline,
  - a white surf lip, with foam lines rolling in to the shore,
  - wet, darker ground right at the water's edge.
- **Ripples are drifting noise** instead of eight global waves, so the crossing stripes seen from afar are gone.

## 2026-11-01: Adaptive detail, rising seas

- **Adaptive detail:** chunks with mountains, rough land or coastline are rebuilt at 2× or 4× resolution when they're big on screen.
  - The extra points are smoothly curved, with added relief and finer coastlines.
  - Builds run in the background, nearest chunks first, within a budget of 40 detailed chunks (10 at 4×).
  - Chunk edges stay crack-free.
  - Nothing changes at the default zoom.
- **Sea level rises with rain:** slightly in rain, and a lot in storms (flooding lowlands). It drains slowly afterwards.

## 2026-11-01: Calmer drag, living water, sharp close-ups

- **Drag speed halved** to 2.2 rad per screen width, with a maximum spin of 18 rad/s. Fling thresholds moved to match: squash from 4 rad/s, clouds from 3.5 rad/s.
- **The atmosphere squashes** along with the planet when flung.
- **Water:**
  - eight travelling waves roll the highlights and glitter in the sun path,
  - a slight swell makes shorelines lap,
  - it reflects the sky at grazing angles,
  - waves get stronger with wind, and the sea turns slate in storms.
- **Terrain surface detail** is drawn per pixel: rock grain, grass speckle and bump lighting, with finer layers fading in as you zoom closer.

## 2026-11-01: Real flinging

- **Spinning moves the world, not the camera.** Drag spins the planet around the axis perpendicular to the drag, in any direction and over the poles. Day and night sweep across it as it turns.
  - Spinning is about twice as fast as before.
  - A hard swipe keeps gliding for several seconds.
  - Touching the planet catches it.
- **Squash and stretch** now uses the planet's actual spin axis, held steady as the spin dies down.
- **Fling effects only trigger on a real fling:** squash above 7 rad/s, clouds above 6 rad/s. Slow or careful drags no longer jiggle anything.
- Pinch and mouse-wheel zoom are handled directly, with easing.

## 2026-11-01: Chunked smooth world, soft clouds, fling

- **4× more detail again:** 655,362 vertices (256 segments per icosahedron edge), with smooth shading instead of flat facets.
- **Chunks:**
  - The planet is now 200 chunks.
  - Sculpting re-uploads only the chunks it touches.
  - Only chunks inside the view and over the horizon are drawn (about two thirds at the default zoom, around 25 close up).
  - Weather recolouring walks the chunks a batch at a time.
- **Clouds:** the sphere puffs are replaced by heaps of soft, shader-lit wisps with frayed edges. They have grey undersides, warm light along the day/night line, near-black storm clouds, and in-cloud lightning. A drifting layer of high cirrus is added.
- **Rain** now falls from the cloud base and is fainter.
- **Fling:** swipe fast and the planet squashes and stretches around its spin axis, and the clouds are thrown outward, then both wobble back.
- **Undo** now records only the vertices each stroke touched, and keeps 30 strokes.
- **Saves:** format v2 stores the gzipped difference from the generated terrain, which is a few KB for a fresh sculpt. Older saves (v1, on the 63- or 127-detail meshes) are carried over by barycentric resampling.
- A loading screen shows while the world builds.

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
