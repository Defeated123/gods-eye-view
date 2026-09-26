# Base44 Dev Environment — God's Eye View

## What this is

God's Eye View is a single Vite dev server (frontend + local API provider
middleware on one origin) that boots **without any API keys** — it falls back
to keyless Esri World Imagery. All keys in `.env.example` are optional and can
be added later via the in-app Provider Settings panel.

## Running it

```
docker compose -f docker-compose.base44.yml up -d --build
```

- `web` (node:24-slim): bind-mounts the repo, runs `npm install` then
  `npm run dev` (Vite) on `0.0.0.0:4173`. Live source is served — edits appear
  via Vite HMR.
- `proxy` (nginx:alpine): host port **3000** → `web:4173`. Strips the dev
  server's `X-Frame-Options` / `Content-Security-Policy: frame-ancestors`
  headers (which would otherwise block the preview iframe) and proxies
  websocket upgrades for HMR.

## Key setup facts

- **No secrets required to boot.** The app starts keyless. Do not call
  `set_secrets` / `generate_development_secrets` unless a real external
  integration (Google 3D Tiles, OpenAI realtime voice, FIRMS, AIS, TomTom,
  OpenSky, etc.) is needed — add those keys in the in-app Provider Settings
  panel instead.
- Node engines require `>=24.14.0`; `node:24-slim` satisfies this. npm does
  not enforce engines (no `.npmrc` / engine-strict).
- `PUPPETEER_SKIP_DOWNLOAD=true` skips the chromium download (puppeteer is a
  devDep for QA scripts only; not needed for the dev server).
- The dev server sets `allowedHosts: true` when `HOST=0.0.0.0`, so the
  rotating sandbox hostname is accepted.
- `node_modules` is an anonymous compose volume (kept out of the bind mount
  for speed; `.gitignore` also ignores it).

## Verifying it works

- `curl -s -o /dev/null -w "%{http_code}" http://localhost:3000/` → `200`
- The served HTML references `/src/main.js` (live source, not a prebuilt
  bundle).
- No `X-Frame-Options` / `Content-Security-Policy` in the proxied response
  headers (nginx strips them).
