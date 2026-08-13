# Altec Wallboard Dashboard

A NOC-style wallboard: `dashboard.html` is a single self-contained file (all
CSS/JS inline, no build step, no framework) that polls several Cloudflare
Worker MCP servers' `/status` (and `/licenses`) routes and renders a
four-zone live view — Network, Tickets & SLA, Security, Business. Open it
directly in any browser, on any TV — `file://` or hosted, doesn't matter.

## Repo split

This repo (`https://github.com/rafouche/Dashboard`) holds only the wallboard
client — it isn't itself an MCP server, just a consumer of one. The workers
it polls (Meraki, Peplink, UniFi, NinjaRMM, HaloPSA, CIPP, Huntress, Pax8,
etc.) live in the separate `https://github.com/rafouche/MCPs` repo, each in
its own `*-mcp/` folder with its own independent `wrangler deploy`.

## Running it

Nothing to install. Just open `dashboard.html` in a browser. All endpoints
point at the `young-math-a33a.workers.dev` subdomain by default — edit the
`ENDPOINTS` object near the top of the `<script>` block if that ever
changes.

### URL params

- `?zone=network|tickets|security|business|all` — which zone(s) to show (default `all`, a 2x2 grid)
- `?demo=1` — force demo data in every zone, ignoring live endpoints
- `?header=0` — hide the top bar (clock/version), useful for a tighter kiosk crop
- `?showInactive=1` — bypass the inactive-client filter (see below) for auditing what's still connected
- `?<endpointKey>=<url>` — override any single endpoint at load time, e.g. `?ninja=http://localhost:8787/status`
- `?<zone>Filter=...` / `?<zone>Sort=...` — set a zone's filter/sort chip on load (also written back to the URL when a chip is clicked, so a kiosk reload keeps its state)

## Zones and their sources

| Zone | Source(s) | Route |
|---|---|---|
| Network | NinjaRMM (individually-tracked devices) | `ninjarmm-mcp.../status` |
| Network | Meraki (org-level device health) | `meraki-mcp.../status` |
| Network | Peplink (org-level device health) | `peplink-mcp.../status` |
| Network | UniFi (org-level device health) | `unifi-mcp.../status` |
| Tickets & SLA | HaloPSA | `halopsa-mcp.../status` |
| Security | Huntress (48h incidents/escalations) | `huntress-mcp.../api/huntress/*` (raw REST passthrough) |
| Security | CIPP (M365 secure score, MFA coverage) | `cipp-mcp.../status` |
| Security | Avanan | not connected yet — placeholder filter slot only |
| Business | Pax8 (subscription renewals) | `pax8-mcp.../api/pax8/*` (raw REST passthrough) |
| Business | Meraki / Peplink (license renewals) | `*-mcp.../licenses` |

Every source degrades independently to demo data if its endpoint is
unreachable — one vendor being down never blanks a whole zone. Meraki,
Peplink, and UniFi network tiles are grouped by client name (with a manual
`ORG_ALIASES` table in the script for cases where a vendor names the same
client differently, e.g. an abbreviation).

## Inactive-client filtering

HaloPSA is treated as the source of truth for whether a client is still
active. `halopsa-mcp`'s `/status` route returns an `inactiveClients` list
(names of Halo clients flagged `inactive`); the dashboard fetches that once
per poll cycle and filters every zone against it by client/org name, since
Meraki/Peplink/Pax8/Huntress/CIPP don't know when we've been offboarded —
only Halo does. Halo's own ticket stats are filtered server-side in
`halopsa-mcp` itself. Filtering only works to the extent a vendor's org name
matches (post-`ORG_ALIASES`-canonicalization) the exact client name in Halo
— if a client still shows up after being marked inactive in Halo, check
whether the vendor's org name actually matches the Halo client record it
should be, and add an `ORG_ALIASES` entry if not. Add `?showInactive=1` to
bypass the filter and see what's still connected.

## Known gaps / TODOs

- Avanan isn't deployed yet — the Security zone's `avanan` filter is a
  placeholder, ready to wire in once `avanan-mcp` exists.
- Field-name assumptions in each worker's `/status` route (ticket fields,
  secure-score/MFA fields, etc.) were confirmed against live payloads at the
  time they were written, but a vendor API change could silently break a
  field mapping — check for `??? / '—'` placeholders on the wallboard if a
  zone stops looking right after a vendor updates their API.
