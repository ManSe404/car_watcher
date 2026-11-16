# Documentation

Car Watch collects the car ads from polovniautomobili.com into a SQLite database on your own machine, through a
Chrome extension and a local Python service, and helps you judge prices. Work continues: open items are in the
[roadmap](roadmap.md).

Read in this order:

| # | Document | What it answers | Status | Last checked |
|---|---|---|---|---|
| 1 | [product/vision.md](product/vision.md) | Why it exists, for whom, what you get, what it is not | current | 2026-11-16 |
| 2 | [product/features/README.md](product/features/README.md) | Every feature, its status and where it lives in the code | current | 2026-11-16 |
| 3 | [guides/setup.md](guides/setup.md) | How to install the host and the extension, run the UI, and what to redo after moving the repository | current | 2026-11-16 |
| 4 | [architecture/overview.md](architecture/overview.md) | How it is built: parts, flows, failures, privacy, security | current | 2026-11-16 |
| 5 | [diagrams/context.md](diagrams/context.md) | Car Watch and the systems around it | current | 2026-11-16 |
| 6 | [diagrams/container.md](diagrams/container.md) | The parts of the extension and the service | current | 2026-11-16 |
| 7 | [diagrams/page-capture.md](diagrams/page-capture.md) | What happens to a page you open | current | 2026-11-16 |
| 8 | [diagrams/collection-flow.md](diagrams/collection-flow.md) | How planned collection runs in the worker tab | current | 2026-11-16 |
| 9 | [reference/services/extension.md](reference/services/extension.md) | The extension: parts, build, fixed values, storage, contracts | current | 2026-11-16 |
| 10 | [reference/services/service.md](reference/services/service.md) | The Python service: its three programs, setup, settings, files, contracts | current | 2026-11-16 |
| 11 | [reference/codebase.md](reference/codebase.md) | Where each part lives in the code | current | 2026-11-16 |
| 12 | [reference/data-model.md](reference/data-model.md) | The SQLite tables and what writes them | current | 2026-11-16 |
| 13 | [reference/api.md](reference/api.md) | Native messages, extension messages, UI routes | current | 2026-11-16 |
| 14 | [reference/configuration.md](reference/configuration.md) | Every `CARWATCH_*` setting | current | 2026-11-16 |
| 15 | [reference/cli.md](reference/cli.md) | The `carwatch` and `carwatch-host` commands | current | 2026-11-16 |
| 16 | [guides/development.md](guides/development.md) | Checks, the dev loop, the contract tests, conventions | current | 2026-11-16 |
| 17 | [roadmap.md](roadmap.md) | Open work in priority order: collection fixes first, then trends and the dashboard | current | 2026-11-16 |
| 18 | [adr/0001-collect-in-the-users-own-browser.md](adr/0001-collect-in-the-users-own-browser.md) | Why collection runs in your own browser from the site's page data | accepted | 2026-11-16 |
| 19 | [adr/0002-native-messaging-instead-of-an-http-service.md](adr/0002-native-messaging-instead-of-an-http-service.md) | Why Chrome starts the host per message instead of a running service | accepted | 2026-11-16 |

**Layout.** `product/` says why and what, `architecture/` how it is built, `diagrams/` holds the Mermaid
diagrams, `reference/` what exists and can be checked against the code (one page per deployable under
`reference/services/`), `guides/` how to do things, `adr/` the decisions, `roadmap.md` what comes next. A change
updates the documents it makes wrong in the same commit.
