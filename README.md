# @pipeworx/usgs-national-map

USGS The National Map (TNM) Access API — the federal download catalog for
national geospatial data: 3DEP elevation (DEMs and lidar point clouds), US Topo
and Historical Topo quadrangles, NHD hydrography, NLCD land cover, National
Structures, Transportation and Boundaries. Every product record carries a
direct download URL, format, footprint and publication date.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1857+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `tnm_datasets(filter?)` — the dataset catalog with each dataset's exact
  title, refresh cycle and formats. Call first.
- `tnm_search_products(datasets?, bbox?, polygon?, prod_formats?, prod_extents?, q?, date_type?, start_date?, end_date?, limit?, offset?)` —
  search downloadable products.
- `tnm_products_at_point(latitude, longitude, datasets?, prod_formats?, buffer_degrees?, limit?)` —
  which products cover a single coordinate.

## Auth

Keyless.

## Data sources

- <https://tnmaccess.nationalmap.gov/api/v1/datasets> — dataset catalog.
- <https://tnmaccess.nationalmap.gov/api/v1/products> — product search.

## Traps

- `datasets` takes the dataset's **full title**, not its id, and a title the
  API does not recognise is **ignored rather than rejected** — you get an
  unfiltered result set that looks like a successful narrow search. Resolve the
  exact string with `tnm_datasets` first.
- `bbox` is `xmin,ymin,xmax,ymax` in WGS84 decimal degrees, **longitude
  first**. A reversed box returns zero items with a 200; this pack refuses one
  rather than passing it through.
- The response's `total` is the real match count and `items` is one page. A
  short array with a large `total` means "page it", not "that is all".

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "usgs-national-map": {
      "url": "https://gateway.pipeworx.io/usgs-national-map/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/usgs-national-map/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 1857+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/tnm_datasets \
  -H 'Content-Type: application/json' \
  -d '{"filter":"elevation"}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/tnm_datasets`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-usgs-national-map` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/usgs-national-map/mcp` endpoint shown above. It needs network access;
there is no offline mode.

```json
{
  "mcpServers": {
    "usgs-national-map": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-usgs-national-map"]
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
ask_pipeworx({ question: "your question about Usgs National Map data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
