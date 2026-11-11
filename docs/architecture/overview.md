# Architecture overview

How Car Watch is built: a Chrome extension that collects, and a Python service on the same computer that stores,
plans and analyses. The diagrams are in [../diagrams/](../diagrams/): [context](../diagrams/context.md),
[containers](../diagrams/container.md), [pages you open](../diagrams/page-capture.md) and
[planned collection](../diagrams/collection-flow.md).

## Context

- **The site.** polovniautomobili.com is a Next.js site. Each page carries its data as JSON (`__NEXT_DATA__` on a
  full load, `/_next/data/{build}/{locale}{path}.json` when you click inside the site). Car Watch reads that JSON
  and nothing else.
- **Your browser.** All requests to the site come from your own Chrome or Chromium session, either because you
  opened a page or from a pinned background tab the extension opens.
- **Your computer.** The service, the database and the UI run locally. Nothing listens on a network port except
  the UI, on 127.0.0.1, while you run it.

## Goals

- Keep the price history of every passenger-car ad on the site, and how long each ad has been listed.
- Show market statistics per model and flag ads listed below a fair price.
- Collect without getting in the site's way: slow, one request at a time, and stop when the site refuses.
- Keep seller data out: only car fields leave the browser.

## Constraints

- Chrome native messaging starts the host program for every message and takes its one reply, so the host
  cannot keep running after it answers. Long work (the fair-price model) runs in a separate, detached process.
- Chrome refuses native-messaging replies over 1 MB and sends at most 64 MiB (`MAX_REPLY_BYTES`,
  `MAX_REQUEST_BYTES` in `host.py`).
- The site stops paging deep result lists, so a full pass is split into price bands (`FULL_BANDS` in `crawl.py`);
  a band with more than `partition_cap` ads is halved.
- The host is registered on Linux only (`carwatch install` exits on other platforms), for Google Chrome and
  Chromium.
- The extension id is pinned by the public `key` in `extension/wxt.config.ts`; the host accepts only that id.

## Building blocks

### Extension (`extension/`, WXT, TypeScript, Zod)

| Part | File | What it does |
|---|---|---|
| Page hook | `entrypoints/page-hook.ts` | Runs in the page's own JavaScript world. Wraps `fetch` and hands a copy of every `/_next/data/...json` response to the content script. Makes no request of its own. |
| Collector | `entrypoints/collector.content.ts` | Content script on the site. Passive mode saves the pages you open; worker mode (URL hash `#carwatch-worker`) runs the planned tasks. |
| Extraction | `lib/extract.ts` | Turns the site's JSON into the capture contract with Zod. Keeps an allow-list of car fields, removes phone numbers and e-mail addresses from descriptions. |
| Worker helpers | `lib/worker.ts` | The data URL for a task, and what a response means (blocked, gone, retry, drift). |
| Badge | `lib/badge.ts` | The box on an ad page: first seen, price changes, days listed. DOM APIs and `textContent` only. |
| Background | `entrypoints/background.ts` | Routes messages from the content script and popup, runs the 10-minute alarm, opens and closes the worker tab. |
| Host client | `lib/service.ts` | One native message per request, 30-second timeout, outbox of up to 100 captures while the host is missing. |
| Popup | `entrypoints/popup/` | Status, the automatic switch, start a search or a full pass, pause 24 hours or resume, send the outbox. |

### Service (`service/src/carwatch/`, Python, SQLite, pydantic, scikit-learn, FastAPI + Jinja)

| Part | File | What it does |
|---|---|---|
| Host | `host.py` | `carwatch-host`: reads one framed message from stdin, validates it, answers on stdout, exits. |
| Contract | `models.py` | The capture shapes shared with `extension/lib/extract.ts`, the work shapes shared with `extension/lib/messages.ts`. Unknown fields are ignored. |
| Store | `store.py` | The SQLite schema and every read and write. |
| Planner | `crawl.py` | Runs, tasks, leases, the price bands, detail pages, the pause and the automatic switch. |
| Valuation | `analysis.py`, `jobs.py` | Market statistics, comparison and the fair-price model; `jobs.py` runs the model in its own process under a lock file. |
| Notifications | `notify.py` | `notify-send` for price drops, at most five per capture, only when `notify-send` exists. |
| UI | `app.py`, `web.py`, `charts.py`, `fmt.py`, `templates/` | Server-rendered pages over the database, an SVG price chart, Serbian number and date formats. |
| Install | `install.py`, `cli.py` | Writes or removes the native-messaging host manifest; the `carwatch` command. |
| Settings | `config.py` | `CARWATCH_*` environment variables, see [configuration](../reference/configuration.md). |

## Runtime

### Pages you open

Every search and ad page you open is saved. On a full page load the content script reads `__NEXT_DATA__`; when
you click inside the site, the page hook passes on the JSON the site's own code fetched. The capture goes to the
background, which sends it to the host as a native message. The host stores it and answers with what it knows
about the ad, which the badge shows. See [page-capture](../diagrams/page-capture.md).

### Planned collection

The host decides what to fetch; the extension only executes it. See
[collection-flow](../diagrams/collection-flow.md).

1. Every 10 minutes (first one minute after the extension starts, and on browser start) the background asks the
   host for its status. If there is work and no worker tab is open, it opens a pinned, inactive tab on the site's
   search page with `#carwatch-worker`.
2. The worker tab reads the site's current build id, then asks for tasks in batches of 10 and fetches each one's
   data URL with the user's own session, waiting `request_gap_seconds` plus up to one second of random jitter
   between requests.
3. Each answer goes back as a capture with its `task_id`. When the first page of a search band is in, the host
   adds the remaining pages (25 ads a page) or halves the band if it holds more than `partition_cap` ads.
4. When no task is left, the tab reports `work-done` and the background closes it.

What the host plans by itself, each time it prepares work:

- **A full pass** over the whole site when the last one started `crawl_interval_days` ago or more. The README
  estimates one pass at about 3,000 search pages; the number depends on the site.
- **Detail pages** (damage, registration, origin, equipment) for active ads seen in the last 10 days that have
  none yet, up to `details_daily_limit` a day, best deals first. They wait until no search page is pending, and
  for up to 15 minutes after a full pass so the fair prices are fresh.
- **Leases.** A task handed out and not reported within `lease_seconds` is handed out again.

### Failures

| Reported as | When | What the host does |
|---|---|---|
| `blocked` | 403, 429, 503, or HTML instead of JSON (a challenge page) | Pauses all collection for `blocked_pause_hours` and returns every leased task to pending. The worker tab stops. Nothing tries to get around it. |
| `retry` | 404 with HTML: the site deployed a new build | Hands the task out again; the tab reloads to learn the new build id, at most three times in ten minutes. |
| `gone` | The ad no longer exists | Marks the task failed and the ad `removed`. |
| `drift` | The JSON changed shape | Counts an attempt; nothing is stored. On a page you opened, the popup shows "Sajt je promenio format". |
| `error` | Anything else | Counts an attempt; a task fails for good after two. |

### The automatic switch and the pause

- The popup's switch is stored in the database (`crawl_state.auto`); until it is first used, `auto_full_runs`
  decides. Off, the host starts no full pass and plans no detail pages, and an automatic pass under way waits.
  Searches you start, and full passes you start with "Pokreni ceo sajt", still run; starting a full pass while an
  automatic one waits claims it as yours.
- "Pauziraj 24 h" stores `crawl_state.paused_until`; while it lasts, nothing is handed out, your own searches
  included. "Nastavi" clears it.

### Fair prices

`carwatch revalue` fits gradient boosting on log price over the latest snapshot (ads seen since the start of the
last finished full pass, or the last 14 days before the first one) with five-fold out-of-fold predictions, so
every ad's expected price comes from a model that never saw that ad. It needs at least 150 ads. It replaces the
`valuations` table. The host starts it as a detached process when a full pass finishes; the UI starts it from
`/runs`, and you can run it by hand. A lock file next to the database keeps it to one at a time.

### The UI

`carwatch ui` serves server-rendered pages over the same database until you stop it. The extension does not use
it. The pages are listed in [the vision](../product/vision.md#what-you-get) and the routes in
[api.md](../reference/api.md).

## Data

One SQLite file (`CARWATCH_DB_PATH`), WAL mode, schema version 3. Tables and what writes them:
[data-model.md](../reference/data-model.md).

## Crosscutting

### Privacy

- **Only car fields.** The allow-list in `extension/lib/extract.ts` picks them. Seller names, seller and dealer ids,
  contact details and profile photos never leave the browser.
- **No contacts in descriptions.** Phone numbers and e-mail addresses are removed from ad descriptions.
- **Nothing outside the contract.** The host ignores any field that `models.py` does not define.
- **Raw captures expire.** When a run finishes, raw page data received more than `capture_retention_days` ago is
  emptied. The parsed ads and sightings stay.
- **The data stays on this machine.** The extension talks to the host over stdin and stdout, and Chrome starts
  the host only for the extension id named in its manifest.

### Security of the UI

- It listens on `CARWATCH_HOST` (127.0.0.1 by default) and refuses any Host header not in `allowed_hosts`, which
  blocks DNS-rebinding pages.
- Every POST (start, cancel, pause, resume, revalue) also checks that the `Origin` header is one of
  `allowed_hosts`, so another site cannot press the buttons.

### Errors

- The host answers `{"ok": false, "error": ..., "retry": true}` when the request was fine but Car Watch failed
  (a locked or unreadable database). The extension keeps such a capture in its outbox; a request the host
  rejects is dropped and shown as the last error in the popup.
- The host logs to stderr at warning level; Chrome forwards it to its own log.

## Earlier version

The original 2023 requests + BeautifulSoup scraper was removed. It stopped working when the site moved to
Next.js behind a Cloudflare challenge. It is still in git history: `git show 2546d05:main.py`.

## Open

- `AdSummary.ui_url` exists in `models.py` and the badge shows a link to the UI when it is set, but nothing sets
  it, so the badge never links to the UI.
- The host is registered on Linux only; macOS and Windows registration is not built.
- The UI is in Serbian only.
