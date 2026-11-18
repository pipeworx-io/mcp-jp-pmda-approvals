# @pipeworx/jp-pmda-approvals

Japan's approved new prescription drugs as the Pharmaceuticals and Medical Devices Agency (PMDA) publishes them: every new-drug and partial-change approval since fiscal year 2018 with brand, applicant, approval date, ingredient and indication, the agency's English index of review reports, its notices revising drug precautions, and its lists of approved medical devices.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 2016+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `jp_pmda_approvals_search(query?, year?, limit?)` — searches the agency's yearly lists of approved new drugs (fiscal years run April to March). Each row has brand name and presentations, applicant and corporate number, approval date, approval type (new product or partial change), review field, the Japanese ingredient name, the indication line and flags such as orphan drug. The lists are Japanese: search with katakana brands, Japanese ingredient names, companies or indication words. `year` is a fiscal year, `recent` (default, latest three) or `all`. Every answer states the fiscal years searched and the latest approval in them.
- `jp_pmda_review_reports_search(query, limit?)` — searches the agency's English index of review reports by English brand name or non-proprietary name. Returns the month approved and the links to the English translation and the Japanese original. Only new-active-ingredient drugs the agency chose to translate are indexed.
- `jp_pmda_safety_search(query?, date_from?, fiscal_year?, limit?)` — searches the agency's notices that revise the precautions for use of a drug (使用上の注意の改訂指示通知): notice date and number, the generic ingredients concerned (Japanese names), and links to the revision text and the investigation report. The wording of each change is in the linked documents, which are not read. Fiscal years 2018 onward; default the three most recent.
- `jp_pmda_device_approvals_search(query?, year?, category?, limit?)` — searches the agency's yearly lists of approved medical devices in four lists (`new`, `improved_clinical`, `improved_no_clinical`, `generic`): brand, generic device-type name, applicant, approval date and type, approval number where listed, and for new and clinical-data improved devices the clinical field, purpose, clinical basis and review period. Default the two most recent fiscal years across all four lists; the list of remanufactured single-use devices is not read.
- `jp_pmda_recalls_search(query?, class?, product_type?, year?, date_from?, date_to?, limit?)` — searches the agency's recall notices (医薬品等の回収に関する情報) for drugs, quasi-drugs, cosmetics, in-vitro diagnostics, medical devices and regenerative products, by fiscal year from 2024 and class (I serious, II temporary or curable, III unlikely harm). Each hit: recall number, class, product type, generic and product name, the marketing authorisation holder, posting date, the agency's remark and the notice link. Default is the current fiscal year; every answer lists the lists read and the dates covered, and a list that cannot be read is reported as unread.
- `jp_pmda_recall_get(recall_number)` — one notice by number (such as `1-2475`): affected lots, quantity and shipping period (truncated when very long), the recall reason as the company states it, the health damage feared, the recall start date and intended use. An unknown number is `found:false`. The notice's contact person and the company's addresses are not returned.

## Auth

Keyless.

## Data sources

- PMDA (医薬品医療機器総合機構), list of approved new drugs (新医薬品の承認品目一覧): read live per call, nothing stored. Fiscal years 2018 onward are readable; older years are published only as legacy spreadsheets or PDF and are reported as unread, never as empty. The current fiscal year is partial.
- PMDA, English index of review reports for new-active-ingredient drugs.
- PMDA, notices of revisions of precautions for drugs (使用上の注意の改訂指示通知): read live per call, one page per fiscal year. Fiscal years 2018 and 2019 carry the posting date rather than the issue date (the answer says which). A year whose page cannot be read is reported as unread.
- PMDA, lists of approved medical devices (新医療機器・改良医療機器・後発医療機器の承認品目一覧): read live per call, one spreadsheet per list per fiscal year, 2018 onward. A year the agency publishes only as a PDF is reported as unread, never as empty.
- PMDA, recall information for drugs, quasi-drugs, cosmetics, medical devices and regenerative products (医薬品等の回収に関する情報): read live per call, one list per fiscal year and class (fiscal 2024 onward) plus one page per notice. Notices are written by the recalling company. A class with no notice has no list page and is named, not guessed; a page that is not a list is reported as unread, never as empty.

Traps worth knowing: the lists carry no English names (brand names are katakana, ingredients are Japanese names), so an English query on the list finds only Latin text; the review-report index is the English route. Neither source has a per-product record page, so there is no get-by-id tool. The package-insert and review-report document search on the agency's site is a script-driven form and is not read. A zero-match answer says which fiscal years were searched; `year=all` widens it.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "jp-pmda-approvals": {
      "url": "https://gateway.pipeworx.io/jp-pmda-approvals/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/jp-pmda-approvals/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 2016+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/jp_pmda_approvals_search \
  -H 'Content-Type: application/json' \
  -d '{"query":"キイトルーダ","year":"all","limit":3}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/jp_pmda_approvals_search`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-jp-pmda-approvals` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/jp-pmda-approvals/mcp` endpoint shown above. It needs network access;
there is no offline mode.

```json
{
  "mcpServers": {
    "jp-pmda-approvals": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-jp-pmda-approvals"]
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
ask_pipeworx({ question: "your question about Jp Pmda Approvals data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
