# Roadmap

Collection, storage, the fair-price model and the UI work end to end (see [the features](product/features/README.md)).
What follows is open, in priority order: first the fixes that keep collection correct and polite to the site, then
the features the weekly passes exist for.

Status: **open** (not started), **idea** (not decided), **done** (kept here until the list is renumbered).

## Paused

Car Watch is **paused** since 2026-11-22: the owner stopped using it, the extension is not loaded, so nothing is
collected and the items below wait. The code, the tests and CI still work. To resume: `pnpm build` in
`extension/`, load `extension/.output/chrome-mv3` unpacked in `chrome://extensions` (the host stays registered;
`uv run carwatch info` in `service/` shows it), open a search on the site and check that the popup shows captures
arriving and that the newest `captured_at` in the database moves.

## Collection correctness

1. **The worker reports a capture the host rejects.** open.
   The worker tab sends each task's capture and ignores the answer (`runTask` in
   `extension/entrypoints/collector.content.ts`). When the host rejects the capture (for example after the contract
   changed on one side only), the task is neither done nor failed: it stays leased, goes back to pending after
   `lease_seconds`, its attempts never grow, and the same page is fetched again on every round while the run never
   finishes. Fix: on a rejected capture send `work-fail` with reason `error`, so the task fails after two attempts;
   when the host cannot be reached, stop the worker. Touches the collector and a first test of the worker loop.
2. **Tell an unrecognised ad page apart from a removed ad.** open.
   For a detail task, `judgeCapture` in `extension/lib/worker.ts` answers `gone` when the page is neither an ad nor
   a search. A change in the site's ad-page data would then mark every fetched ad `removed` instead of showing
   drift, and the task note keeps only `gone 200`, so the two cannot be told apart later. Fix: `drift` for an
   unrecognised page, `gone` only for `notFound`, a redirect or a search page, and the cause in the failure
   message. Touches `worker.ts`, `tests/worker.test.ts`.
3. **Start the valuation only after a full pass.** open.
   The host starts `carwatch revalue` when any run finishes (`host.py`, `Crawler._finish_if_done`), so collecting
   one small search refits the model over the whole snapshot. The commit that introduced it meant a finished full
   pass. Fix: return the run's kind and start the valuation for `full` only; update the host test that uses a
   search run.
4. **CI on pull requests.** done (2026-11-22).
   `.github/workflows/checks.yml` runs the service checks and the extension checks on every pull request; the jobs
   `service` and `extension` are required checks on `master`.
5. **Pin the host's replies like the capture.** open.
   The status, work and summary shapes are written twice, in `service/src/carwatch/models.py` and
   `extension/lib/messages.ts`, with no shared example; a rename on the Python side breaks the popup only at
   runtime. Fix: Python tests write reply fixtures that an extension test reads, as the capture fixtures do.
6. **Numbered schema migrations.** open, before the next schema change.
   `Store` writes `PRAGMA user_version` but never reads it; `_upgrade` knows one column. Fix: read the version and
   run numbered upgrade steps. Touches `store.py`, `tests/test_crawl.py` (the old-database test).

## Market features

7. **Price trends.** open.
   Median price per model over time, one point per full pass, with the count and the average beside the median.
   Needs at least two finished passes; the sightings are already stored. Open: one point per pass or per calendar
   week, and how to show that a change in the mix of cars can look like a trend. Touches `analysis.py` and a UI
   page.
8. **Days on market and sold ads.** open.
   An ad missing from a full pass that covered its price band is gone; days listed run from `first_seen` to its
   last sighting. Open: how to treat bands of a pass that failed or was cancelled. Touches `crawl.py` or
   `analysis.py`, the data model.
9. **One analytics dashboard.** open.
   Brand and model filters (model depends on brand), a production-year range, a period filter (year, month or a
   custom range), brand shares with click-through to that brand's models, average and median price for the
   market, a brand and a model, and trends over time and by production year. A separate program that reads the
   same database and knows nothing about the extension. Open: built on DuckDB or on SQLite queries; whether it
   replaces `carwatch ui` or sits beside it.

## Smaller items

10. **Badge link to the UI.** open. `AdSummary.ui_url` exists and the badge shows a link when it is set, but
    nothing sets it. Open: set it to the UI's address always (the link fails while the UI is not running), or
    remove the field.
11. **Opening `/runs` should not plan work.** open. `Crawler.status` prepares work (reclaims leases, may start a
    full pass and plan detail pages), and the UI's `/runs` page calls it on a GET. Fix: a read-only status for the
    UI.
12. **macOS and Windows.** open. Host registration elsewhere (a manifest folder on macOS; a JSON file, a registry
    key and a launcher on Windows), per-OS paths for the database and settings, and CI on those systems.
13. **Reliability data from an outside source.** idea. No source chosen.
