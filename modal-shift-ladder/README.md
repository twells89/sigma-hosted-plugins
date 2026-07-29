# modal-shift-ladder

A freight **modal-shift opportunity ladder**: one row per lane showing the current
truck vs intermodal split as a stacked bar, ranked so the lanes with the most
convertible truck volume sit on top. Lanes above a configurable truck-share
threshold get flagged as shift candidates. Optional per-lane savings-per-mile and
CO₂-avoided columns render on the right and roll up into the header summary.

Built for freight/3PL dashboards where the question is *"which lanes should move
off the road, and what do we get for it?"* — an operational ranking, not a KPI tile.

**Hosted:** <https://twells89.github.io/sigma-hosted-plugins/modal-shift-ladder/>

## Register it

```bash
curl -sS -X POST "$SIGMA_BASE_URL/v2/plugins" \
  -H "Authorization: Bearer $SIGMA_API_TOKEN" -H 'Content-Type: application/json' \
  -d '{"name":"Modal Shift Ladder","description":"Truck vs intermodal split by lane, ranked by shift opportunity","url":"https://twells89.github.io/sigma-hosted-plugins/modal-shift-ladder/","type":"element"}'
```

`POST /v2/plugins` can return a masked 404 even when it registered — confirm with
`GET /v2/plugins` and take the `pluginId` from there.

## Editor panel / spec config

| Key | Type | Required | Meaning |
|---|---|---|---|
| `source` | element | yes | the data element to read |
| `lane` | column (text) | yes | lane label, e.g. `Chicago, IL → Houston, TX` |
| `truckLoads` | column (number) | yes | loads currently on truck |
| `railLoads` | column (number) | yes | loads currently intermodal |
| `savingsPerMile` | column (number) | no | $ saved per mile if converted |
| `co2Avoided` | column (number) | no | kg CO₂ avoided |
| `sortBy` | dropdown | no | `Opportunity` (default, = truck loads desc) · `Volume` · `CO2 avoided` |
| `maxLanes` | dropdown | no | `5`–`16`, default `8` |
| `flagThreshold` | dropdown | no | flag truck share above this %, default `85` |
| `truckColor` / `railColor` / `flagColor` / `inkColor` | color | no | per-embed theming; defaults are neutral |

Colors are config, not baked in, so the same bundle can be themed per workbook.

In a workbook spec, **every** `config` value is a bare string — column bindings are
bare `columnId` strings (the `{kind:"column",…}` object form is rejected), and even
numeric dropdowns must be quoted:

```json
{ "kind": "plugin", "pluginId": "<pluginId>",
  "config": {
    "source": { "kind": "element", "elementId": "lane-tbl" },
    "lane": "col-lane", "truckLoads": "col-truck", "railLoads": "col-rail",
    "savingsPerMile": "col-spm", "co2Avoided": "col-co2",
    "sortBy": "Opportunity", "maxLanes": "8", "flagThreshold": "85"
  } }
```

## Expected data shape

One row per lane (the plugin also sums duplicate lane rows, so a raw grain works):

| lane | truck_loads | rail_loads | savings_per_mile | co2_avoided_kg |
|---|---|---|---|---|
| `Chicago, IL → Houston, TX` | 8400 | 4700 | 0.42 | 1240000 |
| `Los Angeles, CA → Phoenix, AZ` | 9600 | 820 | 0.55 | 2110000 |

`savings_per_mile` is averaged across duplicate rows weighted by truck loads;
loads and CO₂ are summed.

## Implementation notes

- **React is vendored and loads before the SDK.** `@sigmacomputing/plugin`'s UMD
  build has React as a hard peer dependency — without it the factory throws before
  assigning `client`, leaving `window.SigmaPlugin` with no `client`, and the plugin
  silently serves synthetic data forever. Sigma does not inject React into the
  plugin iframe. Both files are vendored rather than pulled from a CDN so the
  bundle also works when Sigma's server-side render service fetches it.
- **`ResizeObserver` drives a full re-render.** Sigma sizes the panel after the
  first paint, so anything measuring its own container has to redraw on resize.
  The render also fits the visible row count to the available height, and drops the
  right-hand metric column (<560px) then the in-bar percentages (<400px).
- **The "sample data" badge is deliberate.** When there is no client or the
  bindings can't be read, the fallback renders *and says so* — plausible-looking
  synthetic numbers are exactly how a broken column binding hides itself.
- No customer or prospect identifiers in this bundle; the synthetic lanes are
  generic US city pairs.
