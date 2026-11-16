# Extension (`extension/`)

The Chrome extension that collects. It reads the data the site's own pages load, keeps only the car fields and
hands them to the host on the same computer through native messaging. It also runs the work the host plans, in a
pinned background tab. It has no settings of its own and talks to nothing but the site and the host.

Built with [WXT](https://wxt.dev/), TypeScript and Zod; Manifest V3; Chrome and Chromium.

## What runs

| Part | File | Runs in | Does |
|---|---|---|---|
| Background | `entrypoints/background.ts` | the extension's service worker | Routes messages from the content script and the popup, runs the 10-minute alarm, opens and closes the worker tab |
| Host client | `lib/service.ts` | the service worker | One native message per request with a 30-second timeout; the outbox and the popup's counters in extension storage |
| Collector | `entrypoints/collector.content.ts` | a content script on `https://www.polovniautomobili.com/*`, from `document_start` | Passive mode saves the pages you open and shows the badge; worker mode (URL hash `#carwatch-worker`) runs the planned tasks |
| Page hook | `entrypoints/page-hook.ts` | the page's own JavaScript world, injected by the collector | Wraps `fetch` and passes a copy of every `/_next/data/...json` response to the collector; makes no request itself |
| Extraction | `lib/extract.ts` | the collector | Site JSON to the capture contract, allow-list of car fields, removal of phone numbers and e-mail addresses |
| Worker helpers | `lib/worker.ts` | the collector | The data URL of a task and what a response means (`judgeResponse`, `judgeCapture`) |
| Badge | `lib/badge.ts` | the collector | The box on an ad page and the banner in the worker tab, each in a closed shadow root, built with DOM APIs and `textContent` only |
| Popup | `entrypoints/popup/` | the toolbar popup | Status, the automatic switch, collect this search, full pass, pause or resume, send the outbox |

## Build and run

```sh
cd extension
pnpm install                 # also runs `wxt prepare`
pnpm build                   # output: extension/.output/chrome-mv3
```

Load `extension/.output/chrome-mv3` with `chrome://extensions` → Developer mode → Load unpacked. Chrome remembers
that folder: after moving the repository, load it again from the new place. `pnpm dev` runs WXT's development
build without opening a browser (`webExt.disabled` in `wxt.config.ts`). The host has to be registered first, see
[the service](service.md).

Checks: `pnpm test` (Vitest), `pnpm compile` (`tsc --noEmit`), `pnpm build`. There is no lint step.

## Configuration

Nothing is configured by the user. The fixed values live in the code:

| Value | Where | What |
|---|---|---|
| Manifest `key` | `wxt.config.ts` | Pins the extension id to `anfomkmnpkpjpjdlpngmmkedbplleiam`, the only id the host accepts |
| Permissions | `wxt.config.ts` | `storage`, `alarms`, `activeTab`, `nativeMessaging`; `page-hook.js` is web-accessible on the site only |
| `HOST_NAME` | `lib/service.ts` | `io.github.manse404.carwatch`; a Python test checks it matches `carwatch/install.py` |
| `TICK_MINUTES` | `entrypoints/background.ts` | 10: how often the background asks the host for work |
| `TIMEOUT_MS`, `OUTBOX_LIMIT` | `lib/service.ts` | 30 seconds per native message; at most 100 queued captures, the oldest dropped first |
| `WORKER_URL` | `lib/worker.ts` | The search page the worker tab opens, with `#carwatch-worker` |
| Batch size | `nextWork` in `lib/service.ts` | 10 tasks per `work-next` |
| Reload limit | `tooManyReloads` in `entrypoints/collector.content.ts` | At most three reloads in ten minutes when the site's build changes |

The gap between requests comes from the host (`gap_seconds` in each batch); the worker adds up to one second of
random jitter.

## What it stores

| Where | Key | What |
|---|---|---|
| `storage.local` | `stats` | Captures in total and today, the last capture time, the last error (shown in the popup) |
| `storage.local` | `outbox` | Captures waiting for the host |
| `storage.session` | `workerTab` | The id of the open worker tab |
| the worker tab's `sessionStorage` | `carwatch-reloads` | Times of recent reloads, for the reload limit |

## Contracts

- **With the host:** the native messages in [interfaces](../api.md#native-messages-extension-to-host). The request
  and reply types are `HostRequest` and the shapes in `lib/messages.ts`, written by hand to match
  `carwatch/host.py` and `carwatch/models.py`.
- **The capture:** `Capture` in `lib/extract.ts`. The Vitest file snapshots write
  `service/tests/fixtures/capture_*.json`, which the Python tests read, so a change on either side fails a test
  (see [development](../../guides/development.md#the-contract-between-extension-and-service)).
- **Inside the extension:** `Message` in `lib/messages.ts`, sent with `runtime.sendMessage`
  ([interfaces](../api.md#messages-inside-the-extension)).
- **Page hook to collector:** a `carwatch:page-data` `CustomEvent` on the injected script element, whose `detail`
  is a JSON string `{url, text}`.
- **With the site:** the Next.js page data, `__NEXT_DATA__` on a full load and
  `/_next/data/{build}/{locale}{path}.json` otherwise. When its shape changes, extraction fails and the popup
  shows "Sajt je promenio format".

## Tests

`tests/extract.test.ts` (extraction, seller-data removal, the contract snapshots) and `tests/worker.test.ts`
(data URLs, build id, response and capture verdicts), over made-up, site-shaped pages in `tests/fixtures/`. The
background, the host client, the worker loop, the badge and the popup have no tests.
