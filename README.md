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
