# AgentScraper CLI

`agentscraper` fetches live web pages as clean Markdown, text or HTML, straight from your terminal or your agent's shell.
It is the command line client for the [AgentScraper API](https://agentscraper.cc).

## Quick start

```bash
npx @agentscraper-cc/cli login                                   # paste your API key (not echoed)
npx @agentscraper-cc/cli fetch https://example.com -o text       # the page as Markdown
```

Get a key at [agentscraper.cc](https://agentscraper.cc). Node.js 18 or later.

## Install

```bash
npm install -g @agentscraper-cc/cli
agentscraper --help
```

## Use

```bash
agentscraper fetch https://example.com/a https://example.com/b      # up to 10 URLs; JSON with results[] and statuses[]
agentscraper fetch https://example.com -o text                      # just the content (Markdown, else text, else HTML)
agentscraper fetch https://example.com -o html --html               # the page HTML
agentscraper fetch https://example.com --country-code de --render   # from Germany, rendered in a real browser
agentscraper fetch --data @request.json                             # the whole request body from a file (- for stdin)
agentscraper account                                                # plan, credits and rate limit
agentscraper fetch --help                                           # every option, with the API's own descriptions
```

A fetch that is still running after about 100 seconds is collected automatically with the same
`Idempotency-Key`, so it is charged once.

## MCP server

The AgentScraper API is also a remote MCP server (Streamable HTTP) at `https://agentscraper.cc/mcp`, with the tools
`get_page_contents` and `get_account_status`. It takes the same API key as a bearer token.

```bash
claude mcp add --transport http agentscraper https://agentscraper.cc/mcp \
  --header "Authorization: Bearer $AGENTSCRAPER_API_KEY"
```

Any other client: URL `https://agentscraper.cc/mcp`, header `Authorization: Bearer <key>`. Config for Cursor, VS Code and
Claude Desktop: [MCP docs](https://agentscraper.cc/docs/mcp). The server's entry for the MCP Registry is [server.json](server.json)
(`cc.agentscraper/agentscraper`).

## Agent plugin and skill

This repo is also a plugin for Claude Code, Grok Build and Gemini CLI, and holds an agent skill. Each one connects the
MCP server above and teaches the agent when to fetch a page with it or with `agentscraper`.

```bash
npx skills add agentscraper-cc/cli                                    # the skill, for any agent that reads SKILL.md
claude plugin marketplace add agentscraper-cc/cli && claude plugin install agentscraper@agentscraper
gemini extensions install https://github.com/agentscraper-cc/cli
```

Network and credentials: the plugin calls `https://agentscraper.cc/mcp` only. The CLI calls `https://agentscraper.cc/api` and
`https://agentscraper.cc/openapi.json`, and `https://registry.npmjs.org` (`npx` and a once-a-day update check, off with
`AGENTSCRAPER_NO_UPDATE_NOTIFIER=1`). Both need one credential, your AgentScraper API key,
read from `AGENTSCRAPER_API_KEY` or saved by `agentscraper login`. No hooks, no scripts run on install.

## The commands follow the live API

`agentscraper` builds its commands and options from the API's OpenAPI document at
[https://agentscraper.cc/openapi.json](https://agentscraper.cc/openapi.json), cached for an hour. When the API gains a field or an endpoint, the
option or command is there on your next run, without updating the CLI. Offline, it uses the last cached
copy, then the copy bundled with the installed version (`agentscraper spec` shows which one; `--offline` skips the
download). An hourly job here publishes a new version whenever the API spec changed.

## Configuration

| Setting | Option | Environment | Default |
|---|---|---|---|
| API key | `--api-key` | `AGENTSCRAPER_API_KEY` | the key saved by `agentscraper login` (`~/.config/agentscraper/config.json`, mode 0600) |
| API base URL | `--base-url` | `AGENTSCRAPER_API_BASE_URL` | `https://agentscraper.cc/api` |
| API spec URL | `--spec-url` | `AGENTSCRAPER_SPEC_URL` | `https://agentscraper.cc/openapi.json` |
| Output | `-o json\|text\|<field>` | | `json` |
| Update notice | | `AGENTSCRAPER_NO_UPDATE_NOTIFIER=1` turns it off | once a day, terminals only |

Requests and your key only ever go to the base URL, never to a server named in the spec.

## Exit codes

| Code | Meaning |
|---|---|
| 0 | OK |
| 1 | The API answered with an error (JSON on stderr, with its `requestId`), every URL failed with `-o text`, or the fetch was still running at `--max-wait` |
| 2 | Usage: unknown command or option, a value the API does not accept |
| 3 | Network: no answer. The error carries the `Idempotency-Key` to collect the result without a new charge |

## Links

- [agentscraper.cc](https://agentscraper.cc)
- [CLI docs](https://agentscraper.cc/docs/cli)
- [API docs](https://agentscraper.cc/docs)
- [API spec](https://agentscraper.cc/openapi.json)

## License

MIT, © agentscraper.cc
