# Car Watch

Car Watch collects the passenger-car ads from polovniautomobili.com into a SQLite database on your own
machine and helps you judge prices. You get each ad's price history and how long it has been listed, market
statistics per model, and a fair-price model that flags ads listed below what similar cars cost.

It has two parts. A Chrome extension reads the data the site's own pages load (Next.js page data) and keeps
only the car fields. It hands them to Car Watch on your computer through Chrome's
[native messaging](https://developer.chrome.com/docs/extensions/develop/concepts/native-messaging): Chrome
starts a small Python program, the host, for each message, and the host stores the data in SQLite and plans
further collection. Nothing has to be running beforehand and nothing listens on a network port. A web UI over
the same database runs only when you start it. The UI is in Serbian.

For now the host is installed on Linux only (Google Chrome and Chromium).

## Run it

You need [uv](https://docs.astral.sh/uv/) with Python 3.13 or newer, Node and pnpm.

```sh
cd service && uv sync && uv run carwatch install      # registers the host with Chrome and Chromium
cd ../extension && pnpm install && pnpm build         # output: extension/.output/chrome-mv3
```

In Chrome: `chrome://extensions` → Developer mode → Load unpacked → `extension/.output/chrome-mv3`. Open any
search or ad on the site; the popup shows whether captures arrive. `uv run carwatch ui` in `service/` serves the
UI at http://127.0.0.1:8050/ until you stop it.

## Documentation

[docs/README.md](docs/README.md) lists the documents in reading order: what you get and how to control
collection, setup, the architecture with diagrams, the database, the settings and development.

## Personal use only

The site's terms forbid republishing ad content. Car Watch is for personal use: do not publish or share
the data it collects. Car Watch is not affiliated with polovniautomobili.com. It collects slowly from your own
browser session and stops when the site refuses; nothing tries to get around the site's bot protection.

## License

[PolyForm Noncommercial 1.0.0](LICENSE.md). The code is free for learning, research, and personal and hobby
projects, and for educational and research institutions, charities and government bodies. Commercial use
needs a separate agreement with the author.
