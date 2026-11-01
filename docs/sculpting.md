# Sculpting

Tap **Sculpt** to open the edit panel. While it's open, the planet stops auto-spinning.

## Picking

The brush finds the ground with a raycast through a bounding-volume tree (three-mesh-bvh). The tree is refit every 4 frames during a stroke and again when it ends.

## Gestures

- **Drag on the planet** to sculpt.
- **Drag on empty space** to spin the planet.
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

Each stroke saves a snapshot first. Undo (↶) steps back through the last 15 strokes.

## Saving

- The app saves to `localStorage` under `tinyworld-save-v1`. The save holds the seed, the weather and, once you've sculpted, every vertex height as Int16 (about 440 KB).
- It saves 0.4 seconds after a stroke or a weather change, and whenever the app goes to the background.
- A world sculpted on the previous half-resolution mesh (`DETAIL` 63) is carried over when it loads. Old vertices keep their heights, and each new in-between vertex takes the mean of its old neighbours.
- **New world** rolls a new seed. If you've sculpted, it asks first.
