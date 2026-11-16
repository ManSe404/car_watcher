# Service (`service/`)

The Python package `carwatch` (Python 3.13 or newer, managed with uv). It stores what the extension collects,
plans further collection, computes the fair prices and serves the UI. Nothing in it runs all the time: it ships
three programs, each started when needed, over one SQLite file.

| Program | Started by | Lives | Entry point |
|---|---|---|---|
| `carwatch-host` | Chrome, for every native message from the extension | one message: reads it on stdin, answers on stdout, exits | `host.main` |
| `carwatch revalue` | the host when a collection run finishes, the UI's button, or you | until the fair-price model is fitted; one at a time under a lock file | `cli.main` → `jobs.revalue` |
| `carwatch ui` | you | until you stop it; listens on `CARWATCH_HOST:CARWATCH_PORT` (127.0.0.1:8050) | `cli.main` → `app.create_app` |

`carwatch install`, `uninstall` and `info` set up the host registration ([commands](../cli.md)).

## Install and run

```sh
cd service
uv sync
uv run carwatch install      # writes the host manifest for Chrome and Chromium (Linux)
uv run carwatch ui           # the UI, until you stop it
```

The host manifest points at `service/.venv/bin/carwatch-host`, so `carwatch install` has to run again after the
repository moves. A change in `src/carwatch/` reaches the host on its next message; restart `carwatch ui` to see
UI changes.

Checks: `uv run pytest`, `uv run ruff check src tests`, `uv run mypy --strict src`.

## Configuration

`CARWATCH_*` environment variables, all optional, read by `Settings` in `src/carwatch/config.py`; the list is in
[configuration](../configuration.md). Chrome starts the host, so the host sees Chrome's environment, not your
shell's. The host passes its database path on to a valuation it starts. There is no `.env` file.

## What it writes

| What | Where | Written by |
|---|---|---|
| The database | `CARWATCH_DB_PATH` (default under `$XDG_DATA_HOME/carwatch/`) | the host, the UI and the valuation; the schema is created when a `Store` opens ([data model](../data-model.md)) |
| The valuation lock | next to the database, `.valuation.lock` appended to its name | `carwatch revalue`, the UI's valuation |
| The host manifests | `google-chrome/` and `chromium/` `NativeMessagingHosts/` under `$XDG_CONFIG_HOME` or `~/.config` | `carwatch install` |
| Desktop notifications | `notify-send`, when it exists and `CARWATCH_NOTIFY` is on | the host, on price drops |

Logs go to stderr: the host at warning level (Chrome passes them on to its own log), `carwatch ui` and
`carwatch revalue` at info level.

## Contracts

- **Native messages** from the extension: `HostRequest` in `src/carwatch/host.py`; unknown request fields are
  refused, unknown capture fields ignored. Every reply is `{"ok": true, "value": ...}` or
  `{"ok": false, "error": ..., "retry": bool}` and at most 1 MB ([interfaces](../api.md#native-messages-extension-to-host)).
- **The capture** and the work shapes: `src/carwatch/models.py`. The capture is pinned by the shared fixtures in
  `tests/fixtures/`; the work and status shapes are mirrored by hand in `extension/lib/messages.ts`.
- **The database:** `_SCHEMA` in `src/carwatch/store.py`, schema version 3; `_upgrade` adds the one column older
  databases lack. The UI, the host and the valuation share the file (WAL mode, five-second busy timeout).
- **The UI routes:** [interfaces](../api.md#web-ui-routes). Server-rendered HTML; every POST checks the `Origin`
  header and every request the `Host` header against `CARWATCH_ALLOWED_HOSTS`.
- **The host manifest:** `manifest` in `src/carwatch/install.py`, name `io.github.manse404.carwatch`, allowing only
  the extension id pinned in `extension/wxt.config.ts`.

## Inside

| Module | Does |
|---|---|
| `host.py` | Framing, request validation, dispatch to the store and the planner |
| `crawl.py` | Runs, tasks, leases, price bands, detail planning, the pause and the automatic switch |
| `store.py` | The schema, ingest of captures, the reads for the UI |
| `analysis.py`, `jobs.py` | The snapshot, market statistics, comparison, deals, the fair-price model; the detached valuation and its lock |
| `app.py`, `web.py`, `charts.py`, `fmt.py`, `templates/` | The UI |
| `install.py`, `cli.py`, `config.py`, `notify.py`, `models.py` | Registration, the command line, settings, notifications, the shared shapes |

The full map is in [codebase](../codebase.md), the flows in [the architecture](../../architecture/overview.md#runtime).

## Tests

`tests/` runs offline with `uv run pytest`: the host as a function and as a real process with a framed message on
stdin, the store, the planner, the analysis, the install, the UI and the helpers. Every test database is created
under pytest's temporary folder.
