# Sailing

Tap **Sail**, then tap any sea or lake. A small ship launches there and the camera flies down into a third-person chase view. **Planet** flies back out. If you tap land, the ship launches at the nearest water within about 0.15 rad. If there's none, tap again.

## Controls

| Action | Touch | Keyboard |
|---|---|---|
| Steer | Hold ◀ / ▶ | A / D or ← / → |
| Sails (furled → half → full) | Tap the sail button | W / S or ↑ / ↓ |
| Look around the ship | Drag anywhere | |
| Zoom the chase camera | Pinch (0.014 to 0.2) | Mouse wheel |

The panel shows the speed in knots, an arrow for where the wind is blowing relative to your heading (up is ahead), and the point of sail.

## The ship

- 0.009 long (about three-quarters of a base cell), built from simple shapes:
  - a tapered, extruded hull and a deck,
  - a cabin and two masts,
  - two sails that billow and swing with the wind,
  - a red pennant that flutters downwind.
- It lives in the planet's frame (a child of `spin`) as a unit direction plus a heading in the tangent plane, so it turns with the planet's slow spin.
- **Sitting on the sea:** `seaOffset()` in JS mirrors the water shader's swell, wind swell and tide term for term. To make that possible, the noise swell became four slow travelling waves (frequencies 22, 31, 47 and 63; see `SWELL`).
- **Pitch and roll** follow the surface slope, sampled 0.004 ahead, behind and to each side.
- **Heel** comes from the side wind and from turning. Hitting land rocks the ship.

## Speed

- **Wind** is the same trade-wind field as the water (`windAtJS` mirrors the shader's `windAt`).
- **Polar curve** (fraction of top speed by angle off dead downwind): 0° → 0.62, 40° → 0.86, 90° → 1 (beam reach), 120° → 0.88, 138° → 0.5, 150° → 0.12, 180° → 0.04. Pointing into the wind nearly stops you.
- **Target speed** = sail level × 0.028 × wind strength × polar.
  - Wind strength is 0.12 + 0.88 × the local wind (capped at 1.2), × (0.55 + 0.45 × the weather's wind), × 0.6 on lakes.
  - The calm belts around 30° and 60° leave you nearly becalmed. Storms are fastest.
- Speed eases toward the target at 55% per second.
- **Turning:** 0.85 rad/s, down to 40% of that when stopped.
- Knots shown are speed × 900.

## Land

- `groundAt()` finds the ground height under any point: the nearest base vertex, then barycentric interpolation inside the triangle around it that contains the point. It also returns the lake flag.
- Each frame the ship probes 0.65 of a ship length ahead. If the ground there is within 0.0007 of the water surface, the ship stops and rocks.
- If the water drops away under the ship (the sea drains after a storm), it's **aground** and can't move until a tide or rain lifts it.

## Wake

Up to 360 soft foam points (a small custom point shader) are dropped behind the stern in pairs. They drift outward, ride the swell, and fade over 2 to 3.5 seconds. They're spawned faster at higher speed and not at all when stopped.

## Near the surface

These kick in as the camera drops below about 0.3 above the sea, in any mode:

- **Sky:** a shell at radius 1.235, drawn from the inside and added over the background.
  - Blue from horizon to zenith, warm at dusk, with a sun disc and glow.
  - It greys in storms and pales in fog, and dims at night.
  - Stars fade out under a daytime sky.
- **Haze:** scene fog in the horizon colour closes in to about 0.9 away, much closer in fog and rain. Far from the surface the fog sits at 1000 and does nothing.
- The camera's near plane follows its altitude (down to 0.0006), so the ship and nearby water don't clip. The far plane is 60, and the stars sit 30 to 50 away.

## Camera moves

- Flying in and out takes 1.7 s: an eased arc that blends direction and eases the distance geometrically, while the up vector and the look-at point blend too.
- While sailing, the camera trails the ship smoothly. Its up is the local vertical.
