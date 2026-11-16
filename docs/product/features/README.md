# Features

What Car Watch offers today and what is planned, each row checked against the code. Paths are relative to the
repository root; `carwatch/` means `service/src/carwatch/`.

| Feature | What you can do | Status | Where in code | Doc |
|---|---|---|---|---|
| Save the pages you open | Every search and ad page you open on the site is stored with its prices; nothing has to be started | shipped | `extension/entrypoints/collector.content.ts` (passive mode), `extension/entrypoints/page-hook.ts`, `extension/lib/extract.ts`, `carwatch/host.py`, `carwatch/store.py` | [page capture](../../diagrams/page-capture.md) |
| Ad badge | On an ad page, a box shows when Car Watch first saw the ad, its price changes and the days it has been listed | shipped | `extension/lib/badge.ts`, `Store.summary` in `carwatch/store.py` | [overview](../../architecture/overview.md#pages-you-open) |
| Badge link to the UI | Open the ad in the Car Watch UI from the badge | partial: the badge shows the link only when `AdSummary.ui_url` is set, and nothing sets it | `extension/lib/badge.ts`, `AdSummary` in `carwatch/models.py` | [roadmap](../../roadmap.md) |
| Weekly full pass | The host plans a pass over every passenger-car ad, split into price bands, and a pinned background tab runs it | shipped | `carwatch/crawl.py`, `extension/entrypoints/background.ts`, `extension/entrypoints/collector.content.ts` (worker mode), `extension/lib/worker.ts` | [collection flow](../../diagrams/collection-flow.md) |
| Ad detail pages | Damage, registration, origin and equipment for up to `details_daily_limit` ads a day, best deals first | shipped | `Crawler._plan_details` in `carwatch/crawl.py` | [overview](../../architecture/overview.md#planned-collection) |
| Collect one search | "Pokupi ovu pretragu" in the popup fetches every page of the search open in the tab | shipped | `extension/entrypoints/popup/main.ts`, `Crawler.start_search_run` in `carwatch/crawl.py` | [interfaces](../../reference/api.md) |
| Control collection | Turn automatic collection off and on, pause for 24 hours, resume, start a full pass now (popup or `/runs`) | shipped | `extension/entrypoints/popup/main.ts`, `carwatch/host.py`, `Crawler.set_auto`, `Crawler.pause`, `Crawler.start_full_run`, `carwatch/web.py` | [vision](../vision.md#how-you-control-collection), [overview](../../architecture/overview.md#the-automatic-switch-and-the-pause) |
| Stop when the site refuses | A 403, 429, 503 or a challenge page pauses all collection for `blocked_pause_hours` | shipped | `judgeResponse` in `extension/lib/worker.ts`, `Crawler.fail` in `carwatch/crawl.py` | [overview](../../architecture/overview.md#failures) |
| Outbox | Captures made while the host is missing or failing wait in extension storage (up to 100) and are sent later | shipped | `extension/lib/service.ts` | [overview](../../architecture/overview.md#errors) |
| Seller data kept out | Only car fields leave the browser; phone numbers and e-mail addresses are removed from descriptions; raw page data expires | shipped | `extension/lib/extract.ts`, `Crawler._finish_if_done` in `carwatch/crawl.py` | [overview](../../architecture/overview.md#privacy) |
| Price-drop notifications | A desktop notification when a stored ad's price drops, through `notify-send` | shipped | `carwatch/notify.py` | [configuration](../../reference/configuration.md) |
| Ads list | `/`: every stored ad, with filters (text, brand, model, search, price, year, mileage, dropped, active) and sorts | shipped | `ads_page` in `carwatch/app.py`, `Store.list_ads` | [interfaces](../../reference/api.md#web-ui-routes) |
| Ad page | `/ads/{id}`: price step chart, details, every sighting | shipped | `ad_page` in `carwatch/app.py`, `carwatch/charts.py` | [interfaces](../../reference/api.md#web-ui-routes) |
| Market statistics | `/market`: per brand and model, median price with quartiles, year, mileage, power, yearly loss of value, number of good deals | shipped | `market_page` in `carwatch/web.py`, `analysis.market` | [interfaces](../../reference/api.md#web-ui-routes) |
| Compare models | `/compare`: median price by production year and what a budget buys, for up to six models | shipped | `compare_page` in `carwatch/web.py`, `analysis.compare`, `carwatch/charts.py` | [interfaces](../../reference/api.md#web-ui-routes) |
| Fair price and deals | `carwatch revalue` fits a fair-price model; `/deals` lists the ads furthest below their expected price | shipped | `analysis.run_valuation`, `carwatch/jobs.py`, `analysis.deals`, `deals_page` in `carwatch/web.py` | [overview](../../architecture/overview.md#fair-prices) |
| Searches you opened | `/searches`: each search you opened yourself, with how many ads it returned | shipped | `searches_page` in `carwatch/app.py`, `Store.list_searches` | [interfaces](../../reference/api.md#web-ui-routes) |
| Collection page | `/runs`: progress of each run; start, cancel, pause, resume, recompute fair prices | shipped | `runs_page` and the POST routes in `carwatch/web.py` | [interfaces](../../reference/api.md#web-ui-routes) |
| Register the host | `carwatch install`, `uninstall` and `info` for Google Chrome and Chromium on Linux | shipped | `carwatch/install.py`, `carwatch/cli.py` | [commands](../../reference/cli.md) |
| Other systems | Register the host on macOS and Windows | planned | none yet | [roadmap](../../roadmap.md) |
| Price trends | Median price per model over time, built from the weekly passes | planned | none yet | [roadmap](../../roadmap.md) |
| Days on market and sold ads | Ads missing from a later pass counted as gone, with how long they were listed | planned | none yet | [roadmap](../../roadmap.md) |
| Analytics dashboard | One dashboard with brand and model filters, a year range, a period filter, brand shares, average and median prices and trends | planned | none yet | [roadmap](../../roadmap.md) |

## Collection

The extension collects in two ways. Passive capture saves the search and ad pages you open, and the badge on an ad
page shows what is already stored. Planned collection is decided by the host: a full pass over the site every
`crawl_interval_days` and detail pages for the best deals, which a pinned background tab fetches one request at a
time from your own session. The popup and the `/runs` page control it, and collection stops by itself when the
site refuses. The outbox keeps captures while the host cannot take them, and only car fields ever leave the
browser.

## Analysis and the UI

`carwatch ui` serves server-rendered pages over the same database while you run it: the stored ads, one ad with
its price history, market statistics per model, a comparison of chosen models, the deals list and the collection
runs. The fair prices behind `/deals` and the good-deal counts on `/market` come from `carwatch revalue`, which
also runs on its own when a collection run finishes.

## Setup

`carwatch install` registers the host with Chrome and Chromium on Linux; the extension is loaded unpacked. Other
operating systems are planned.

## Planned

Trends over time, days on market and the analytics dashboard need data from more than one full pass; the
database already keeps every sighting with its time. The order and the open questions are in the
[roadmap](../../roadmap.md).
