# Weather

Weather is local for now. The six buttons pick a preset, and every value eases toward the new preset over about a second. The current weather is saved.

| Preset | Cloud cover | Cloud grey | Sun | Rain | Snow | Storm | Fog | Wind |
|---|---|---|---|---|---|---|---|---|
| Clear | 0.14 | 0 | 2.7 | 0 | 0 | 0 | 0 | 1 |
| Cloudy | 0.62 | 0.22 | 2.0 | 0 | 0 | 0 | 0 | 1.2 |
| Rain | 0.82 | 0.5 | 1.35 | 1 | 0 | 0 | 0 | 1.4 |
| Storm | 1 | 0.78 | 0.75 | 1.8 | 0 | 1 | 0 | 2.2 |
| Snow | 0.72 | 0.12 | 1.7 | 0 | 1 | 0 | 0 | 0.8 |
| Fog | 0.3 | 0.18 | 1.5 | 0 | 0 | 0 | 1 | 0.6 |

The atmosphere colour and strength also change per preset. It turns greyer and dimmer in storms, and paler and thicker in fog.

## Clouds

- 56 clusters of 16 to 30 soft "wisps" each. A wisp is a camera-facing sprite drawn by a custom shader, and all of them are one instanced mesh.
- Each cluster is a flat-bottomed heap, stretched east–west.
- The shader frays each wisp's edges with two layers of a slowly drifting noise texture. Its shading comes from:
  - its height in the heap (grey underneath, bright on top),
  - which side of the planet it's on (dark at night, with a warm tint at the day/night line),
  - the weather's greyness, reaching near-black slate in storms,
  - lightning, which lights the clouds from inside.
- Clusters orbit eastward at altitude 1.17 to 1.22. Each has its own speed, multiplied by the preset's wind value, and a slow north–south wobble.
- Each cluster has a random rank. It fades in when its rank is below the current cover value and fades out when it isn't. Wisps also swell up to 30% in grey weather.

## Cirrus

High streaky cirrus sits on a shell at radius 1.255. Its texture is generated at load from noise stretched east–west and broken into patches. The shell drifts eastward and is drawn additively, so it brightens the sky and never darkens it. Its opacity per preset:

| Clear | Cloudy | Rain | Storm | Snow | Fog |
|---|---|---|---|---|---|
| 0.55 | 0.45 | 0.2 | 0.08 | 0.3 | 0.25 |

## Rain and snow

- Precipitation only falls from clusters that are mostly faded in. Each drop belongs to a cloud and falls from its base straight down toward sea level, drifting along with its cloud.
- **Rain:** up to 2,600 short, faint streaks (1,300 × the rain value). Streaks are longer, faster and more slanted in storms.
- **Snow:** up to 1,600 soft flakes that fall slowly and sway.

## Storms

When the storm value is above 0.5, lightning strikes every 1.2 to 4.7 seconds. There's a 30% chance of a quick double strike. Each strike:

- draws a jagged bolt from a random cloud to the surface,
- flashes the sky light,
- lights up the clouds from inside.

## Fog

A pale shell sits just above sea level (radius 1.032) and fades in to 60% opacity. Lowlands disappear under it while mountains poke through.

## Accumulation

- **Snow** settles while it's snowing (+0.06 per second, up to 1) and melts otherwise (−0.035 per second). At full cover the snowline drops by 0.1, so snow reaches the lowlands.
- **Wetness** builds while rain is above 0.4 (+0.25 per second) and dries at −0.05 per second.
- Terrain is recoloured at most every 0.15 seconds while either value is changing.
