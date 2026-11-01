# Roadmap

## Real weather

Drive the presets from Apple WeatherKit instead of the buttons.

- WeatherKit's REST API needs a JWT signed with a key from the Apple Developer account. That key must stay server-side, so the plan is a small proxy (a Lambda or a Cloudflare Worker) that signs the token and returns the current conditions.
- The page gets the player's location from the browser's geolocation API, calls the proxy, and maps the condition codes onto blends of the presets (cover, rain, snow, wind and so on) rather than snapping to one.
- WeatherKit requires the Apple Weather mark and a link to its data sources wherever the data is shown.
- Keep the buttons as a manual override.

## Ideas

- Time of day: put the sun where it really is for the player's local time.
- Brushes: paint biomes directly (sand, forest, snow), plus a "noise" brush for rough ground.
- Rivers and lakes that form after rain.
- Little trees and houses scattered on grassland.
- Ambient sound for rain, wind and thunder.
- Show temperature and wind on the world itself.
- A native wrapper (Capacitor) if home-screen widgets or the native WeatherKit framework become worth it.
