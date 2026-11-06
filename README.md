# Verfy MCP Lock

Fail the pull request when an MCP tool your coding agents use changed since you locked it.

An MCP server can change a tool's description or input schema in any release, and that text goes straight into your agent's context. This action reads every tool your MCP config serves, compares each definition with the SHA-256 pins in your committed `verfy.lock`, and fails the check if one changed, with the diff in the log and as an annotation on the pull request.

It runs the [Verfy CLI](https://github.com/anvarbekovjavohir277-dot/verfy) (MIT). Website and docs: https://verfy.dev

## Use it

First lock today's tools on your machine and commit the lock:

```bash
pip install verfy
verfy lock check --update        # shows each tool definition, then writes verfy.lock
git add verfy.lock && git commit -m "Lock MCP tools"
```

Then add the check:

```yaml
name: Verfy MCP Lock
on: [pull_request, push]
permissions:
  contents: read
jobs:
  lock:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v7
      - uses: anvarbekovjavohir277-dot/verfy-action@v1
```

Exit codes: 0 all tools match, 1 a tool is new or gone, 2 a tool changed, 3 a server did not answer.

## Badge

Running this check? You can say so in your README:

[![Locked by Verfy](https://verfy.dev/badge/locked-by-verfy.svg)](https://verfy.dev/docs#badge)

```markdown
[![Locked by Verfy](https://verfy.dev/badge/locked-by-verfy.svg)](https://verfy.dev/docs#badge)
```

The badge states one fact, which you put there yourself: this repository commits `verfy.lock` and its CI fails when a tool definition changes. It is not a security review or a guarantee that the tools are safe.

## Inputs

| Input | Default | What it is |
|---|---|---|
| `config` | `.mcp.json` | MCP config whose servers are checked |
| `lock` | `verfy.lock` | the committed lock file |
| `servers` | all | space-separated server names to check |
| `version` | `0.16.0` | Verfy CLI version from PyPI |
| `python-version` | `3.13` | Python that runs Verfy (3.10 or newer) |

## Remote servers and keys

Stdio servers are started from their command, as your MCP client would; pin package versions. Remote servers (a `url` in the config) are called over Streamable HTTP with the headers in the config. Write a key as `${MCP_TOKEN}` and pass it from a secret:

```json
{ "mcpServers": { "docs": { "type": "http", "url": "https://mcp.example.com/mcp", "headers": { "Authorization": "Bearer ${MCP_TOKEN}" } } } }
```

```yaml
      - uses: anvarbekovjavohir277-dot/verfy-action@v1
        env:
          MCP_TOKEN: ${{ secrets.MCP_TOKEN }}
```

Verfy does not follow redirects, so a key only goes to the address in the config, and it never prints header values. Servers that only speak the older SSE transport are not supported yet.

## Limits

A passing check means the definitions match what someone reviewed and locked, not that they are safe. A server can serve different definitions to different clients, so keep the lock on developers' machines too. More in [SECURITY_LIMITATIONS.md](https://github.com/anvarbekovjavohir277-dot/verfy-releases/blob/main/docs/SECURITY_LIMITATIONS.md). Report a vulnerability or a way past Verfy privately through the [security policy](https://github.com/anvarbekovjavohir277-dot/verfy-releases/blob/main/SECURITY.md).

MIT license. The Verfy name and logo are covered by the [trademark policy](https://github.com/anvarbekovjavohir277-dot/verfy-releases/blob/main/TRADEMARKS.md).
