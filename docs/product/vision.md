# Vision

## Why

Judging a used-car price from one ad is guesswork. The site shows today's price, not how it moved, how long the
car has been listed, or what similar cars cost. Car Watch keeps every passenger-car ad from polovniautomobili.com
in a database on your own machine, so those questions get answers: each ad's price history and days listed,
market statistics per model, and a fair-price model that flags ads listed below what similar cars cost.

## For whom

One person looking for a car, on their own computer, with Chrome or Chromium on Linux. The UI is in Serbian,
like the site.

## What you get

`carwatch ui` serves these pages until you stop it:

| Page | What it shows |
|---|---|
| Oglasi `/` | Every stored ad, with filters: price drop, price, year, mileage… |
| `/ads/{id}` | One ad: price chart, details, every sighting |
| Tržište `/market` | Per brand and model: median price, year, mileage, power, yearly loss of value |
| Ponude `/deals` | Ads listed below the model's expected price |
| Uporedi `/compare` | Median price by year, and what a budget buys, for the models you pick |
| Pretrage `/searches` | The searches you ran, with how many ads each one returned |
| Povlačenje `/runs` | Collection progress; start, pause, resume or cancel a run |

On the site itself, a small box on every ad page shows what Car Watch already knows about that ad, and a desktop
notification tells you when a stored ad drops in price.

## How you control collection

The popup shows whether captures arrive and what the worker tab is doing.

- **"Isključi automatsko" / "Uključi automatsko"** turns automatic collection off and on. While it is off, Car
  Watch starts no full pass and plans no ad pages by itself, and an automatic pass under way waits and continues
  from the same place when you turn it back on. What you start yourself still runs: "Pokupi ovu pretragu" collects
  the search open in the tab, and "Pokreni ceo sajt" runs a full pass now (it also takes over a waiting automatic
  pass). A new install starts with automatic collection on; `CARWATCH_AUTO_FULL_RUNS=false` makes it start off.
- **"Pauziraj 24 h"** stops all collection for a day, your own searches included. "Nastavi" resumes.
- **"Pošalji red"** sends captures that waited in the outbox while Car Watch was not reachable.

## What it is not

- **Not a scraper that fights the site.** It reads only the data the site's own pages load, from your own browser
  session, one request every 3 to 4 seconds, and stops for 12 hours when the site refuses. Nothing tries to get
  around the site's bot protection.
- **Not a way to republish ads.** The site's terms forbid republishing ad content. Car Watch is for personal use:
  do not publish or share the data it collects. It is not affiliated with polovniautomobili.com.
- **Not a store of people.** Seller names, ids, contact details and photos never leave the browser.
- **Not a server.** Nothing has to run beforehand and nothing listens on a network port except the UI on
  127.0.0.1, while you run it.
- **Not for commercial use** without a separate agreement: the licence is PolyForm Noncommercial 1.0.0.
