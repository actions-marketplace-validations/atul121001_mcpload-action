# mcpload MCP load test (GitHub Action)

Load and soak test your remote MCP server in CI, and fail the build when it gets slower, starts erroring, or leaks memory or sessions.

This is the GitHub Marketplace entry for [mcpload](https://github.com/atul121001/mcpload), a free, open-source load-testing tool for MCP servers built on k6. It runs the same action as `atul121001/mcpload/action`.

## What it does

- Simulates AI agents: opens sessions, lists tools, and calls several tools at once.
- Measures speed (p50/p95/p99) and errors **for each tool**, and fails the job if a tool goes over its budget.
- Optional soak test that watches your server's memory, sessions and open files, and flags leaks.
- Warns when the load generator itself couldn't keep up, so slow CI runners aren't blamed on your server.
- Posts a pass/fail summary on the pull request and uploads the full HTML report.

## Usage

```yaml
jobs:
  mcp-load:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pull-requests: write        # only needed for the PR comment
    steps:
      - uses: actions/checkout@v4
      - run: docker compose up -d --build      # start your MCP server
      - uses: atul121001/mcpload-action@v1
        with:
          url: http://localhost:8080/mcp
          duration: 2m
          vus: '10'
          p95-ms: '800'
          p99-ms: '2000'
          err-rate: '0.01'
          comment-on-pr: 'true'
```

### Soak test for leaks

```yaml
      - uses: atul121001/mcpload-action@v1
        with:
          url: http://localhost:8080/mcp
          soak-min: '15'
          sampler: prometheus
          prom-url: http://localhost:8080/metrics
```

At least 10 minutes of steady load is recommended for finding slow leaks.

### If your server needs a login token

```yaml
        with:
          url: https://staging.example.com/mcp
          env: |
            MCP_TOKEN=${{ secrets.MCP_TOKEN }}
```

Only test servers you own or have permission to test.

## Inputs and outputs

All inputs and outputs are listed in [action.yml](action.yml). The most used ones:

| Input | What it does |
|---|---|
| `url` | Your MCP endpoint (required) |
| `duration`, `vus` | How long to run, and how many simulated agents |
| `p95-ms`, `p99-ms`, `err-rate` | Speed and error budgets, applied to every tool |
| `soak-min`, `sampler`, `prom-url` | Soak test and server memory sampling |
| `env` | Extra settings, one `KEY=value` per line (tokens, `TOOL_MIX`, …) |
| `baseline-branch`, `fail-on-regression` | Compare each tool with the last run on that branch (e.g. `main`) and fail on a regression |
| `comment-on-pr` | Post the summary on the pull request |
| `fail-on` | `fail` (default) fails the job on a failed check; `never` only reports |

Outputs include `passed`, `result` (`pass`, `fail` or `error`) and `html-path`.

## Learn more

Full documentation, the report format and the command-line tool: [github.com/atul121001/mcpload](https://github.com/atul121001/mcpload).

## License

[Apache-2.0](LICENSE)
