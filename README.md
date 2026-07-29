# sigma-hosted-plugins

Persistent public host (GitHub Pages) for Sigma custom-plugin bundles used
during migration-skill development and live acceptance testing.

Sigma's plugin `url` field must be reachable both by a human's browser AND
by Sigma's server-side render service — a bare `localhost` URL or an
ephemeral tunnel (ngrok free tier, etc.) does not satisfy that. This repo
exists solely to give each plugin bundle a stable `https://twells89.github.io/sigma-hosted-plugins/<plugin>/`
URL via GitHub Pages (source: `main` branch, `/` root).

## Plugins

- `gauge/` — radial value-vs-target gauge (`@sigmacomputing/plugin`), used by
  the `sigma-plugin-authoring` skill's `recreate-as-plugin` disposition.
- `gauge-v2/` — same gauge with React + the SDK vendored locally.
- `modal-shift-ladder/` — freight modal-shift opportunity ladder: truck vs
  intermodal split per lane, ranked by convertible volume, with optional
  savings-per-mile and CO₂-avoided columns. See
  [`modal-shift-ladder/README.md`](modal-shift-ladder/README.md).

**Vendor React, don't CDN it.** `@sigmacomputing/plugin`'s UMD build has React as
a hard peer dependency: with React absent the factory throws before assigning
`client`, so `window.SigmaPlugin` exists with no `client` and the plugin silently
falls back to its synthetic data. Sigma does not inject React into the plugin
iframe. `gauge-v2/` and `modal-shift-ladder/` both vendor `react.production.min.js`
and `sigmacomputing-plugin.umd.js` alongside `index.html`.
