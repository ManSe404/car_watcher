# Interfaces

Three interfaces: the native messages between the extension and the host, the messages inside the extension, and
the routes of the web UI.

## Native messages (extension to host)

Each message is one JSON object with a `type`, framed as Chrome native messaging frames it: a 32-bit length in
native byte order, then the JSON. The requests are `HostRequest` in `service/src/carwatch/host.py`, mirrored in
`extension/lib/messages.ts`; unknown request fields are refused.

| `type` | Fields | Answer `value` |
|---|---|---|
| `status` | | `version`, `ads`, `captures`, `work` (`WorkStatus`: has work, automatic switch, pause end, latest run's progress, next full pass, detail pages planned today and the limit) |
| `capture` | `capture` (`Capture` in `models.py`) | `IngestResult`: capture id, ads seen, new ads, price changes, and for an ad page the `AdSummary` the badge shows |
| `work-next` | `limit` (1 to 50, default 10) | `WorkBatch`: tasks (`id`, `kind` search or detail, `path`, `params`), `gap_seconds`, `paused_until` |
| `work-fail` | `task_id`, `failure` (`reason`: blocked, gone, retry, drift or error; `status`; `message`) | `{"ok": true}` |
| `start-run` | `kind` (`full` or `search`), `page_url` (required for a search) | `run_id` |
| `pause` | `hours` (0 clears the pause) | `{"ok": true}` |
| `auto` | `enabled` | `{"ok": true}` |

Every answer is `{"ok": true, "value": ...}` or `{"ok": false, "error": "...", "retry": bool}`. `retry` is true
when the request was valid but Car Watch failed, so the extension keeps a capture for later.

The capture contract has two sides that must agree: `extension/lib/extract.ts` writes it and
`service/src/carwatch/models.py` reads it. `service/tests/fixtures/capture_*.json` are the shared examples (see
[development](../guides/development.md#the-contract-between-extension-and-service)).

## Messages inside the extension

The content script and the popup send these to the background with `runtime.sendMessage` (`Message` in
`extension/lib/messages.ts`): `capture`, `drift`, `status`, `flush`, `work-next`, `work-fail`, `work-done`,
`start-run`, `pause`, `auto`. Most are passed on to the host; `drift` and `flush` stay in the extension, and
`work-done` closes the worker tab.

## Web UI routes

Served by `carwatch ui` (`service/src/carwatch/app.py`, `web.py`). Pages are HTML.

| Method and path | What it is |
|---|---|
| `GET /` | Oglasi: every stored ad, with filters (`q`, `brand`, `model`, `search`, `price_min`, `price_max`, `year_min`, `km_max`, `dropped`, `active`) and a sort |
| `GET /ads/{ad_id}` | One ad: price chart, details, every sighting |
| `GET /market` | Tržište: per brand and model, median price, year, mileage, power, yearly loss of value |
| `GET /compare` | Uporedi: median price by year and what a `budget` buys, for the models in `m` |
| `GET /deals` | Ponude: ads listed below the model's expected price |
| `GET /searches` | Pretrage: the searches you opened, with how many ads each returned |
| `GET /runs` | Povlačenje: collection progress and buttons |
| `POST /runs/start` | Start a full pass and clear the pause |
| `POST /runs/{run_id}/cancel` | Cancel a run |
| `POST /runs/pause` | Pause collection for 24 hours |
| `POST /runs/resume` | Clear the pause |
| `POST /analysis/run` | Recompute the fair prices in the background |
| `GET /api/docs`, `GET /api/openapi.json` | FastAPI's generated description of these routes |

Every POST checks the `Origin` header against `allowed_hosts` and redirects back to `/runs`.
