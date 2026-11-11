# Codebase map

Two folders, one per side of the native-messaging boundary. Generated folders (`extension/.output/`,
`extension/.wxt/`, `node_modules/`, `.venv/`) are not committed.

```text
extension/                        Chrome extension (WXT, TypeScript, Zod), built with pnpm
  wxt.config.ts                   manifest: name, public key that pins the extension id, permissions
  entrypoints/
    background.ts                 background service worker: message router, 10-minute alarm, worker tab
    collector.content.ts          content script on the site: passive capture and worker mode
    page-hook.ts                  script injected into the page: copies the site's Next.js data responses
    popup/                        index.html, main.ts, style.css: the toolbar popup
  lib/
    extract.ts                    site JSON to capture contract; allow-list of car fields; contact scrubbing
    service.ts                    native messaging to the host; outbox and stats in extension storage
    messages.ts                   types of the internal messages and of the host requests and replies
    worker.ts                     data URL of a task; how to read a response
    badge.ts                      the box on an ad page
    format.ts                     number and date formatting
  tests/
    extract.test.ts               extraction, seller-data removal, the contract snapshots
    worker.test.ts                data URLs, build id, response and capture verdicts
    fixtures/                     made-up, site-shaped page data
service/                          Python package carwatch (uv, Python 3.13 or newer)
  pyproject.toml                  dependencies, scripts carwatch and carwatch-host, ruff, mypy, pytest
  src/carwatch/
    host.py                       entry point carwatch-host: framing, request validation, dispatch
    cli.py                        entry point carwatch: install, uninstall, info, ui, revalue
    config.py                     Settings from CARWATCH_* variables
    models.py                     the capture contract and the work shapes
    store.py                      SQLite schema, ingest, reads for the UI
    crawl.py                      runs, tasks, leases, price bands, detail planning, pause, automatic switch
    analysis.py                   snapshot, market statistics, comparison, deals, fair-price model
    jobs.py                       runs the valuation in a detached process under a lock file
    notify.py                     notify-send on price drops
    install.py                    host manifest, extension id from the public key
    app.py                        FastAPI app, the ads, ad and searches pages
    web.py                        market, compare, deals and runs pages, and the POST actions
    charts.py                     SVG step chart of an ad's price
    fmt.py                        Serbian number and date formatting
    templates/                    Jinja templates of the pages
  tests/                          pytest: host (also as a real process), store, crawl, analysis, install, UI, helpers
    fixtures/capture_*.json       the contract examples written by the extension's tests
docs/                             this documentation
```

Entry points: `carwatch-host` (`host.main`), `carwatch` (`cli.main`), and in the extension the three WXT
entrypoints plus the popup.
