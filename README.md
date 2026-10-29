# @pipeworx/einpresswire

Live press releases from EIN Presswire's public news sitemap — headline,
publish time, keyword tags and geo location, newest first.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1856+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `einpresswire_latest_releases(since_hours?, limit?)` — the current rolling
  window of releases, newest first. `newest_item_at` tells a caller whether
  the wire is quiet or stalled.
- `einpresswire_search_releases(query, limit?)` — keyword/company/place match
  over headline, keyword tags and geo location, within the current window
  only (there is no body excerpt in this source).
- `einpresswire_get_release(url)` — one specific release, matched by the URL
  a prior call returned. Fails clearly (not silently empty) if the release is
  older than roughly 48 hours.

## Auth

Keyless. No account, no clickthrough terms.

## Data sources

- <https://www.einpresswire.com/sitemap-news> — a Google-News-format XML
  sitemap EIN Presswire itself lists as a `Sitemap:` entry in its
  `robots.txt` (published for crawling, no account, no key). Verified live
  2026-09-21: HTTP 200, ~780 KB, 705 releases spanning roughly the trailing
  48 hours.

**Why this pack does NOT read RSS**, unlike its two sibling wire packs: EIN
Presswire's RSS is gated behind a free account. `/all-rss` is an HTML
directory page, and every category link it lists
(`/subscribe-rss/<category>`) 302-redirects to `/login`. "Requires an
account" is explicitly out of scope under the standing no-login sourcing
rule (logging in to scrape public data is not the same as scraping already-
public data). The sitemap is the genuinely public, keyless alternative for
this publisher — denser and fresher than the gated RSS would have been.

Because this is a sitemap and not RSS/Atom, the shared `parseFeed` parser
(`@pipeworx/shared`, built for `<item>`/`<entry>` blocks) does not apply here
— this pack has its own small `<url>`-block extraction. It does reuse
`canonicalUrl` from `@pipeworx/shared` for URL dedupe, the same helper the
RSS-based sibling packs use.

No structured issuer field is exposed by this source — the issuer is usually
the subject of the headline. No body excerpt either; only headline, keyword
tags, geo location and publish time.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "einpresswire": {
      "url": "https://gateway.pipeworx.io/einpresswire/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/einpresswire/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 1856+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/einpresswire_latest_releases \
  -H 'Content-Type: application/json' \
  -d '{}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/einpresswire_latest_releases`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-einpresswire` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/einpresswire/mcp` endpoint shown above. It needs network access;
there is no offline mode.

```json
{
  "mcpServers": {
    "einpresswire": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-einpresswire"]
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
ask_pipeworx({ question: "your question about Einpresswire data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
