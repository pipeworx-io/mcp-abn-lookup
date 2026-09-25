# @pipeworx/abn-lookup

BYOK. Australian Business Register lookup: resolve an ABN or ACN to the registered
legal entity name, entity type, ABN status, GST registration, address state/postcode,
and business names, or search the register by entity/business name.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1683+ live data sources.

## Tools

- `abn_lookup(abn, _apiKey)` — ABN -> legal name, entity type, status, GST, address, business names.
- `abn_search(name, state?, maxResults?, _apiKey)` — entity/business name -> ranked ABN matches.
- `acn_lookup(acn, _apiKey)` — ACN (company number) -> the same detail as `abn_lookup`.

## Auth

BYO only. There is no platform key. Every call requires the caller's own ABR
web-services GUID via `_apiKey`. Register free at
https://abr.business.gov.au/Tools/WebServices — ABR requires accepting its Web
Services Agreement (terms + a personal-info-sharing acknowledgement) under a
registered identity before it emails a GUID; that's a human consent step, so
Pipeworx does not hold or front one.

Without `_apiKey`, every tool refuses with "requires an API key" and points to
the registration URL above.

## Data sources

- `https://abr.business.gov.au/json/AbnDetails.aspx` — ABN or ACN detail.
- `https://abr.business.gov.au/json/AcnDetails.aspx` — ACN detail.
- `https://abr.business.gov.au/json/MatchingNames.aspx` — name search.

## Gotchas

- The JSON endpoints are JSONP: the body is wrapped as `<callback>({...})`,
  where `<callback>` follows whatever `callback=` query param is sent
  (defaults to `callback` if omitted — verified live). This pack strips the
  wrapper generically by function name, not by hardcoding `callback`.
- An invalid or unrecognised GUID returns **HTTP 200** with every core field
  empty and a `Message` string (e.g. `"The GUID entered is not recognised as a
  Registered Party"`), not an HTTP error. ABR uses the same shape for a
  genuine not-found result, so an empty core field alongside a `Message`
  cannot be told apart from a rejected key by the response alone — this pack
  says so explicitly in the error rather than guessing.
- Field names in the mapped output (`Abn`, `AbnStatus`, `EntityName`,
  `BusinessName[]`, etc.) come from ABR's published JSON sample URLs; the
  *populated* (successful-lookup) shape is unverified end-to-end because no
  fleet GUID exists (BYOK, ruled 2026-09-03). The raw ABR payload is always
  returned as `raw` so a caller with a real key is never blocked by a stale
  field name here.
- `state` on `abn_search` is a client-side filter over ABR's returned
  matches (by each match's `State` field), not a documented ABR query
  parameter — ABR's public JSON sample docs don't show a state filter param
  for `MatchingNames.aspx`.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "abn-lookup": {
      "url": "https://gateway.pipeworx.io/abn-lookup/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/abn-lookup/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 1683+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

This pack takes your own API key (`_apiKey`) — we don't front one for it, so there's no curl here that would run without it. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/abn_lookup`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Standalone (no gateway account)

This package also runs as a local stdio MCP server — no Pipeworx account, no
gateway round-trip:

```json
{
  "mcpServers": {
    "abn-lookup": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-abn-lookup"]
    }
  }
}
```

Or run it directly to confirm it starts:

```bash
npx -y @pipeworx/mcp-abn-lookup
```

It speaks MCP over stdin/stdout and answers `initialize`/`tools/list`/`tools/call`
for **only** this pack's tools — none of the shared meta-tools the gateway
connection above adds. Same source, same tools, no ask_pipeworx routing.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Abn Lookup data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
