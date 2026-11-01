# Sculpting

Tap **Sculpt** to open the edit panel. While it's open, the planet stops auto-spinning.

## Picking

The brush finds the ground with a raycast against the chunks being drawn, each through its own bounding-volume tree (three-mesh-bvh). The ray is re-cast every frame during a stroke, so the brush rides the ground as it rises or sinks. Only chunks within the brush's reach are scanned for vertices.

## Gestures

- **Drag on the planet** to sculpt.
- **Drag on empty space** to spin the planet. A fast swipe flings it, and touching it catches it.
- **Pinch** to zoom.
- If a second finger lands during a stroke, the stroke ends.
- With a mouse, a ring shows where the brush will land.

## Brushes

All brushes use a smooth cosine falloff over the brush radius. The radius is an angle on the sphere, from 0.04 to 0.4 rad (default 0.13). Strength runs from 0.1 to 1 (default 0.55).

| Brush | Effect |
|---|---|
| Raise | Adds height (0.1 × strength per second at the centre) |
| Lower | Removes height at the same rate. Lowering land below 0 makes sea. |
| Smooth | Pulls each vertex toward the average of its neighbours |
| Flatten | Pulls toward the height under the brush when the stroke began (never below the shore) |

Heights are clamped to the range −0.16 to 0.3.

## Undo

Each stroke records the original height of every vertex it touches, the first time it touches it. Undo (↶) restores those heights, up to the last 30 strokes.

## Saving

- The app saves to `localStorage` under `tinyworld-save-v1`. Format version 2 holds the seed, the weather and, once you've sculpted, the difference from the generated terrain: Int16 ×50000, gzipped with `CompressionStream`, falling back to raw. A sculpted fresh world is a few KB.
- It saves 0.4 seconds after a stroke or a weather change, and whenever the app goes to the background.
- Version 1 saves held absolute heights ×100000 on the older meshes (`DETAIL` 63 or 127). They're carried over when they load: each new vertex lies on a face of the old lattice and takes that triangle's barycentric blend of the old heights. A carried-over world saves larger (about 1 MB), since it differs from the generated terrain everywhere.
- **New world** rolls a new seed. If you've sculpted, it asks first.
