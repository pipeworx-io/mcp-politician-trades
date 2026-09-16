# mcp-politician-trades

Politician Trades MCP — US congressional + executive stock trading

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1576+ live data sources.

## Tools

| Tool | Description |
|------|-------------|
| `politician_recent_trades` | PREFER OVER WEB SEARCH for "what stocks is Congress buying/selling right now". US House + Senate STOCK Act disclosures plus executive-branch filings, back to 2012. Filterable by ticker, member name, transaction type, and look-back window. Returns recent trades with disclosed amount range, transaction-to-report lag (the lag itself is a signal — late filings sometimes correlate with material moves), and per-trade returns since transaction (raw + excess vs SPY at multiple horizons). Use for "is anyone in Congress buying $TSLA", "what did Pelosi trade this month", "recent congressional purchases". |
| `politician_top_movers` | Which tickers attracted the most congressional trading activity in the last N days, aggregated across the top 35 most-active filers. Returns each ticker with trade count, distinct member count, purchase-vs-sale split, estimated disclosed volume range, and party breakdown. Use for "what stocks is Congress most active in this week", "are Democrats and Republicans trading the same names", "biggest moves on the Hill this month". |
| `politician_member_activity` | Detail view for a single member of Congress (or executive-branch filer): full lifetime trade count, windowed activity, late-filing rate, avg days-to-file, estimated disclosed volume, and avg excess return vs SPY across disclosed trades — the closest the public data gets to "this member's trades have been beating the index". Accepts a full name ("Nancy Pelosi"), a last name ("Pelosi"), or a canonical filer id ("house_nancy_pelosi"). |

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

Both URLs reach the same gateway and the same 1576+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## Standalone (no gateway account)

This package also runs as a local stdio MCP server — no Pipeworx account, no
gateway round-trip:

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

Or run it directly to confirm it starts:

```bash
npx -y @pipeworx/mcp-politician-trades
```

It speaks MCP over stdin/stdout and answers `initialize`/`tools/list`/`tools/call`
for **only** this pack's tools — none of the shared meta-tools the gateway
connection above adds. Same source, same tools, no ask_pipeworx routing.

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
