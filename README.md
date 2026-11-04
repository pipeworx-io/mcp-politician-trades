# @pipeworx/politician-trades

US congressional stock-trade disclosures under the STOCK Act.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1866+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `politician_recent_trades(ticker?, member?, transaction?, days?, limit?)`,
  `politician_top_movers(days?, limit?)`, `politician_member_activity(member, days?)`
  — House and Senate trades from an open aggregator (the 35 most active filers
  for cross-member views; full history for a named member or ticker).
- `house_ptr_search(member?, ticker?, tx_type?, from?, to?, late_only?, limit?)`
  — House trades parsed from the Clerk of the House's own Periodic Transaction
  Reports, **every member**, with transaction date, notification date, real
  filing date, `days_to_file`, `is_late` (filed more than 45 days after the
  trade, the STOCK Act limit), the disclosed amount bracket and a link to the
  Clerk's document. The window is by FILING date.
- `house_ptr_late_filers(from?, to?, min_transactions?, limit?)` — the whole
  House ranked by late transactions over a filing-date window.

## Auth

Keyless to the caller. The `house_ptr_*` tools read a stored copy; the store is
loaded by `scripts/load-house-ptr.py` (migration 269).

## Data sources

- Clerk of the House of Representatives, financial-disclosure filings
  (Periodic Transaction Reports), 2019 onward. Public, served without
  credentials. Parsed from the filers' own documents; each row links to its
  source document.
- Open aggregator of House and Senate disclosures (the first three tools).

## Things the next person would otherwise rediscover

- **Every `house_ptr_*` response carries a `notice`:** 5 U.S.C. 13107 bars using
  these reports for any commercial purpose other than news and communications
  media dissemination (also credit ratings and solicitation). The notice is the
  caller's warning; the store is built on Bruce's authorisation.
- **Refresh:** `scripts/load-house-ptr.py` is incremental by DocID. The daily
  workflow is proposed at `proposals/agent-config/.github/workflows/house-ptr-refresh.yml`
  (push-blocked path, applied by Bruce). It exits red when the store is behind
  an active Clerk index for 7+ days.
- **Unread filings are listed, not OCR-read:** `coverage.most_recent_unread_filings`
  gives the ten newest with the Clerk document link.

- **House only for `house_ptr_*`. The Senate is not in the store.** The Senate's
  search site requires accepting a use agreement before any disclosure can be
  read, so it is held until that agreement is accepted by the company.
- **Some filings cannot be read.** Scanned filings (no text layer) and a few
  layouts the parser does not read are stored as filings with zero rows and
  counted in `coverage.filings_not_machine_readable`. A member with such a
  filing may have more trades than shown. Before 2019 the layout varies and
  coverage is partial; the store starts at 2019.
- **Amounts are brackets.** `amount_max_usd` is null for open-ended brackets
  ("Over $1,000,000"). The midpoint is only computed for closed brackets.
- **`is_late` is `days_to_file > 45`**, filing date minus transaction date. The
  `days_from_notification` field gives the other reading of the rule.
- Loader refresh: `python3 scripts/load-house-ptr.py --years 2026` skips filings
  already stored, so a daily run is one index download plus the new PDFs. A
  filing-count mismatch between parsed rows and the document's own status
  markers is recorded in `house_ptr_filings.status_markers`.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "politician-trades": {
      "url": "https://gateway.pipeworx.io/politician-trades/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/politician-trades/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 1866+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/politician_recent_trades \
  -H 'Content-Type: application/json' \
  -d '{"ticker":"TSLA","days":30,"limit":50}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/politician_recent_trades`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-politician-trades` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/politician-trades/mcp` endpoint shown above. It needs network access;
there is no offline mode.

```json
{
  "mcpServers": {
    "politician-trades": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-politician-trades"]
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
ask_pipeworx({ question: "your question about Politician Trades data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
