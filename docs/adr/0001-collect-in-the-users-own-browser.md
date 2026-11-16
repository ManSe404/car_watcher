# 1. Collect in the user's own browser from the site's page data

Date: 2026-11-06

## Status

Accepted. Introduced with commit `2e8254e` ("replace the 2023 scraper with Car Watch").

## Context

The 2023 version was a requests + BeautifulSoup scraper. It stopped working when polovniautomobili.com moved to
Next.js behind a Cloudflare challenge. The site's terms forbid republishing ad content.

## Decision

A Chrome extension collects. It reads only the Next.js page data the site's own pages load (`__NEXT_DATA__` on a
full load, `/_next/data/{build}/{locale}{path}.json` on a click), from the pages the user opens and from a pinned
tab that fetches the same JSON same-origin in the user's session. It waits seconds between requests and stops for
hours when the site refuses. Nothing tries to get around the site's bot protection. Only an allow-list of car
fields leaves the browser; phone numbers and e-mail addresses are removed from descriptions. The data is for
personal use and is not published.

## Consequences

- Collection happens only while Chrome runs with the extension loaded.
- A new site build or a change in the page data shape stops extraction until `extension/lib/extract.ts` is
  updated; the extension reports it as drift instead of storing wrong data.
- A refusal pauses all collection (`blocked_pause_hours`); there is no fallback that requests the site another way.
- Seller data never reaches the database, so the analysis cannot use it.
