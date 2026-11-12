# Context Retrieval CLI

`contextretrieval` fetches live web pages as clean Markdown, text or HTML, straight from your terminal or your agent's shell.
It is the command line client for the [Context Retrieval API](https://contextretrieval.net).

## Quick start

```bash
npx @contextretrieval-net/cli login                                   # paste your API key (not echoed)
npx @contextretrieval-net/cli fetch https://example.com -o text       # the page as Markdown
```

Get a key at [contextretrieval.net](https://contextretrieval.net). Node.js 18 or later.

## Install

```bash
npm install -g @contextretrieval-net/cli
contextretrieval --help
```

## Use

```bash
contextretrieval fetch https://example.com/a https://example.com/b      # up to 10 URLs; JSON with results[] and statuses[]
contextretrieval fetch https://example.com -o text                      # just the content (Markdown, else text, else HTML)
contextretrieval fetch https://example.com -o html --html               # the page HTML
contextretrieval fetch https://example.com --country-code de --render   # from Germany, rendered in a real browser
contextretrieval fetch --data @request.json                             # the whole request body from a file (- for stdin)
contextretrieval account                                                # plan, credits and rate limit
contextretrieval fetch --help                                           # every option, with the API's own descriptions
```

A fetch that is still running after about 100 seconds is collected automatically with the same
`Idempotency-Key`, so it is charged once.

## MCP server

The Context Retrieval API is also a remote MCP server (Streamable HTTP) at `https://contextretrieval.net/mcp`, with the tools
`get_page_contents` and `get_account_status`. It takes the same API key as a bearer token.

```bash
claude mcp add --transport http contextretrieval https://contextretrieval.net/mcp \
  --header "Authorization: Bearer $CONTEXTRETRIEVAL_API_KEY"
```

Any other client: URL `https://contextretrieval.net/mcp`, header `Authorization: Bearer <key>`. Config for Cursor, VS Code and
Claude Desktop: [MCP docs](https://contextretrieval.net/docs/mcp). The server's entry for the MCP Registry is [server.json](server.json)
(`net.contextretrieval/contextretrieval`).

## Agent plugin and skill

This repo is also a plugin for Claude Code, Grok Build and Gemini CLI, and holds an agent skill. Each one connects the
MCP server above and teaches the agent when to fetch a page with it or with `contextretrieval`.

```bash
npx skills add contextretrieval-net/cli                                    # the skill, for any agent that reads SKILL.md
claude plugin marketplace add contextretrieval-net/cli && claude plugin install contextretrieval@contextretrieval
gemini extensions install https://github.com/contextretrieval-net/cli
```

Network and credentials: the plugin calls `https://contextretrieval.net/mcp` only. The CLI calls `https://contextretrieval.net/api` and
`https://contextretrieval.net/openapi.json`, and `https://registry.npmjs.org` (`npx` and a once-a-day update check, off with
`CONTEXTRETRIEVAL_NO_UPDATE_NOTIFIER=1`). Both need one credential, your Context Retrieval API key,
read from `CONTEXTRETRIEVAL_API_KEY` or saved by `contextretrieval login`. No hooks, no scripts run on install.

## The commands follow the live API

`contextretrieval` builds its commands and options from the API's OpenAPI document at
[https://contextretrieval.net/openapi.json](https://contextretrieval.net/openapi.json), cached for an hour. When the API gains a field or an endpoint, the
option or command is there on your next run, without updating the CLI. Offline, it uses the last cached
copy, then the copy bundled with the installed version (`contextretrieval spec` shows which one; `--offline` skips the
download). An hourly job here publishes a new version whenever the API spec changed.

## Configuration

| Setting | Option | Environment | Default |
|---|---|---|---|
| API key | `--api-key` | `CONTEXTRETRIEVAL_API_KEY` | the key saved by `contextretrieval login` (`~/.config/contextretrieval/config.json`, mode 0600) |
| API base URL | `--base-url` | `CONTEXTRETRIEVAL_API_BASE_URL` | `https://contextretrieval.net/api` |
| API spec URL | `--spec-url` | `CONTEXTRETRIEVAL_SPEC_URL` | `https://contextretrieval.net/openapi.json` |
| Output | `-o json\|text\|<field>` | | `json` |
| Update notice | | `CONTEXTRETRIEVAL_NO_UPDATE_NOTIFIER=1` turns it off | once a day, terminals only |

Requests and your key only ever go to the base URL, never to a server named in the spec.

## Exit codes

| Code | Meaning |
|---|---|
| 0 | OK |
| 1 | The API answered with an error (JSON on stderr, with its `requestId`), every URL failed with `-o text`, or the fetch was still running at `--max-wait` |
| 2 | Usage: unknown command or option, a value the API does not accept |
| 3 | Network: no answer. The error carries the `Idempotency-Key` to collect the result without a new charge |

## Links

- [contextretrieval.net](https://contextretrieval.net)
- [CLI docs](https://contextretrieval.net/docs/cli)
- [API docs](https://contextretrieval.net/docs)
- [API spec](https://contextretrieval.net/openapi.json)

## License

MIT, © contextretrieval.net
