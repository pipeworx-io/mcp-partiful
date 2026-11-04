# @pipeworx/partiful

Public event listings from Partiful's explore pages — titles, times,
locations, host names, and aggregate RSVP counts for the parties, pop-ups,
panels and meetups Partiful surfaces as crashable.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1867+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `partiful_explore(region?)` — browse the public explore page. Pass a
  region code (`nyc`, `la`, `sf`, `dc`, `chi`, `bos`, `atx`, `mia`) for a
  richer set of themed sections for that city; omit it for the top-level
  cross-city trending overview (NYC/LA/SF only).
- `partiful_event(event_id)` — full public detail for one event: title,
  description, start/end time, location, image, host display names, and
  aggregate RSVP counts. Never returns a guest list or contact info.

## Auth

Keyless. No account, login, or API key of any kind — the explore page and
event pages are rendered for anonymous visitors.

## Data sources

- Partiful, public event listings — the explore carousels and per-event
  pages Partiful serves to any visitor, logged in or not.

## Notes

- Partiful server-renders `/explore` and its per-event pages as a Next.js app
  and ships the full public-field event data inside a `__NEXT_DATA__` JSON
  blob in the plain HTML. No separate API call, no JS execution needed — this
  pack parses that blob directly.
- `/explore` with no region carries only a small baked NYC/LA/SF trending
  overview (~5 events per city). The per-region explore page is a separate,
  much richer per-city page with up to a dozen themed sections (Halloween
  Parties, Pop-Ups & Markets, Meet Someone New, etc.) — use it when you want
  real coverage for one city.
- Only eight region codes resolve on that route (probed live 2026-10-13):
  `nyc`, `la`, `sf`, `dc`, `chi`, `bos`, `atx`, `mia`. Every other candidate
  tried (chicago, seattle, london, philly, houston, san-diego, denver,
  atlanta, nashville, portland, toronto, …) 404s. There is no broader
  city/geo query parameter — this is the whole surface Partiful exposes.
- Guest lists are never exposed, structurally: an unauthenticated request to
  an event page gets `guest: null` from Partiful itself, so there is nothing
  to leak even if this pack tried to surface it. Host display names (shown
  publicly on the event page to any visitor) are returned; host photos,
  phone numbers, emails, and RSVP-form answers are never surfaced.
- `location` normalizes Partiful's two shapes: `structured` (neighborhood,
  display name, address lines, a Google Maps link) and `freeform` (a single
  address string). A private/unlisted event, or an id that never existed,
  both 404 cleanly — `partiful_event` raises a clear error rather than
  returning a half-filled record.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "partiful": {
      "url": "https://gateway.pipeworx.io/partiful/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/partiful/mcp` returns the tools in the table
above **plus the shared Pipeworx meta-tools** — `ask_pipeworx`,
`discover_tools`, `search_within`, `remember`/`recall` and the rest of the
gateway-wide set. So the tool count you see is larger than this table: a
single-pack endpoint currently lists roughly 30 shared tools alongside the
pack's own. The connection's `initialize` response states its exact scope, and
is the authoritative answer for a given day.

This is deliberate, not multiplexing by accident. The meta-tools are what let a
scoped connection answer a question this pack does not cover — via
`ask_pipeworx`, which routes across the whole catalog — without you adding a
second MCP server. There is currently no way to mount a pack endpoint without
them; if the extra schemas cost you more context than the routing is worth,
connect to the full gateway once rather than to several pack endpoints.

Or connect to the full Pipeworx gateway to get every pack's tools listed
directly, instead of just this one's:

```json
{
  "mcpServers": {
    "pipeworx": {
      "url": "https://gateway.pipeworx.io/mcp"
    }
  }
}
```

Both URLs reach the same gateway and the same 1867+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/partiful_explore \
  -H 'Content-Type: application/json' \
  -d '{"region":"la"}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/partiful_explore`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-partiful` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/partiful/mcp` endpoint shown above. It needs network access;
there is no offline mode.

```json
{
  "mcpServers": {
    "partiful": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-partiful"]
    }
  }
}
```

Without a key it calls anonymously, capped at 50 calls a day. A free account
raises that to 200 a day: get a key at https://pipeworx.io/signup and add
`"env": { "PIPEWORX_API_KEY": "<your key>" }` to the server entry.
The key is sent as a Bearer token, the same way a hosted connection sends it,
so calls are metered and billed exactly as they are there.

It lists **only** this pack's tools by default. Set `PIPEWORX_META_TOOLS=1` to
also list `ask_pipeworx` and the other gateway-wide tools.

Behind an HTTPS proxy: Node's built-in `fetch` does not read `HTTPS_PROXY` by
default. Newer Node releases honour it when `NODE_USE_ENV_PROXY=1` is set; check
your Node version's documentation.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Partiful data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
