# Configuration

Car Watch reads `CARWATCH_*` environment variables, all optional (`Settings` in
`service/src/carwatch/config.py`). The host is started by Chrome, so it sees Chrome's environment, not your
shell's: a variable you export in a terminal reaches `carwatch ui` and `carwatch revalue`, but the host only if
Chrome was started with it. When the host starts a valuation, it passes its own database path on.

| Variable | Default | Meaning |
|---|---|---|
| `CARWATCH_HOST` | `127.0.0.1` | Address the web UI listens on |
| `CARWATCH_PORT` | `8050` | Port of the web UI |
| `CARWATCH_DB_PATH` | `$XDG_DATA_HOME/carwatch/carwatch.db`, else `~/.local/share/carwatch/carwatch.db` | Database file |
| `CARWATCH_ALLOWED_HOSTS` | `127.0.0.1`, `localhost` | Host headers the UI answers to, and origins its buttons accept |
| `CARWATCH_NOTIFY` | `true` | Desktop notifications on price drops |
| `CARWATCH_TIMEZONE` | `Europe/Belgrade` | Time zone of the dates in the UI and of the day the detail limit counts |
| `CARWATCH_CRAWL_INTERVAL_DAYS` | `7` | Days between full passes |
| `CARWATCH_AUTO_FULL_RUNS` | `true` | Starting position of the popup's automatic switch; once you use the switch, its choice wins |
| `CARWATCH_DETAILS_DAILY_LIMIT` | `600` | Ad pages fetched per day for details |
| `CARWATCH_DETAILS_PRICE_MAX` | none | Only ads at or below this price get details |
| `CARWATCH_DETAILS_YEAR_MIN` | none | Only ads from this year on get details |
| `CARWATCH_REQUEST_GAP_SECONDS` | `3.0` | Gap between two requests to the site, before up to one second of jitter |
| `CARWATCH_PARTITION_CAP` | `9500` | Most ads in one price band before it is halved |
| `CARWATCH_LEASE_SECONDS` | `600` | A task not reported back within this time is handed out again |
| `CARWATCH_BLOCKED_PAUSE_HOURS` | `12` | How long collection waits after the site refuses |
| `CARWATCH_CAPTURE_RETENTION_DAYS` | `30` | Raw page data older than this is emptied when a run finishes |

`CARWATCH_ALLOWED_HOSTS` is a list: pydantic-settings reads list-like settings from the environment as JSON, so
it is written as a JSON array of host names.
