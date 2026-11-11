# Documentation

Car Watch collects the car ads from polovniautomobili.com into a SQLite database on your own machine, through a
Chrome extension and a local Python service, and helps you judge prices.

Read in this order:

| # | Document | What it answers | Status | Last checked |
|---|---|---|---|---|
| 1 | [product/vision.md](product/vision.md) | Why it exists, for whom, what you get, what it is not | current | 2026-11-11 |
| 2 | [guides/setup.md](guides/setup.md) | How to install the host and the extension and run the UI | current | 2026-11-11 |
| 3 | [architecture/overview.md](architecture/overview.md) | How it is built: parts, flows, failures, privacy, security | current | 2026-11-11 |
| 4 | [diagrams/context.md](diagrams/context.md) | Car Watch and the systems around it | current | 2026-11-11 |
| 5 | [diagrams/container.md](diagrams/container.md) | The parts of the extension and the service | current | 2026-11-11 |
| 6 | [diagrams/page-capture.md](diagrams/page-capture.md) | What happens to a page you open | current | 2026-11-11 |
| 7 | [diagrams/collection-flow.md](diagrams/collection-flow.md) | How planned collection runs in the worker tab | current | 2026-11-11 |
| 8 | [reference/codebase.md](reference/codebase.md) | Where each part lives in the code | current | 2026-11-11 |
| 9 | [reference/data-model.md](reference/data-model.md) | The SQLite tables and what writes them | current | 2026-11-11 |
| 10 | [reference/api.md](reference/api.md) | Native messages, extension messages, UI routes | current | 2026-11-11 |
| 11 | [reference/configuration.md](reference/configuration.md) | Every `CARWATCH_*` setting | current | 2026-11-11 |
| 12 | [reference/cli.md](reference/cli.md) | The `carwatch` and `carwatch-host` commands | current | 2026-11-11 |
| 13 | [guides/development.md](guides/development.md) | Checks, the dev loop, the contract tests, conventions | current | 2026-11-11 |

**Layout.** `product/` says why and what, `architecture/` how it is built, `diagrams/` holds the Mermaid
diagrams, `reference/` what exists and can be checked against the code, `guides/` how to do things. A change
updates the documents it makes wrong in the same commit.
