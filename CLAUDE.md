# Tiny World: notes for Claude

A tiny procedural planet with live weather, built in Three.js. Uri develops it from his phone, so Claude does the editing, committing and pushing.

- The whole app is `index.html` (Three.js 0.187 via an import map from jsDelivr, plain DOM UI). No build step.
- Pushing to `main` deploys to GitHub Pages: https://senzaiken.github.io/tiny-world/
- **Build stamping:** install the hook once per clone: `cp tools/pre-commit .git/hooks/pre-commit && chmod +x .git/hooks/pre-commit`. It writes a new `BUILD` id into `index.html` and `version.json` on every commit touching the app, so open copies show the "new version" banner. Don't edit `BUILD` by hand.
- **Keep the wiki current.** Every change to a system or number updates the matching page in `docs/` and adds an entry to `docs/changelog.md` in the same commit.
- Don't break saves (`localStorage` key `tinyworld-save-v1`). Heights are saved as Int16 (metres ×100000) only after the world is sculpted. If `DETAIL` changes, old sculpts are dropped (the seed still loads); note that in the changelog.
- Weather is local-only for now: buttons drive presets in `PRESETS`. The plan is to feed real WeatherKit data through a small server-side proxy (the JWT signing key must never ship in the page).
- Before pushing: run a headless render (Chromium + `npm i three@0.187.0`, routing the jsDelivr URLs to `node_modules` since the CDN is blocked in the sandbox). `window.__tw` exposes `setWeather`, `heights` and `sculptAt` for tests.
