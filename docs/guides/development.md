# Development

## Checks

```sh
cd service && uv run pytest && uv run ruff check src tests && uv run mypy --strict src
cd extension && pnpm test && pnpm compile && pnpm build
```

`pnpm compile` is `tsc --noEmit`; `pnpm test` is `vitest run`. There is no CI: run both lines before a commit.

## The loop

- **Service.** The host is a new process for every message, so a change in `service/src/carwatch/` takes effect
  on the next message; no restart. Restart `carwatch ui` to see UI changes.
- **Extension.** `pnpm build`, then reload the extension in `chrome://extensions`. `pnpm dev` runs WXT's
  development mode; `wxt.config.ts` turns off WXT's own browser launch (`webExt.disabled`), so load the unpacked
  output yourself.
- **Moving the repository** changes the path of `carwatch-host`: run `uv run carwatch install` again.

## What the tests cover

- The host's tests also start it as a real program with a framed message on stdin, the way Chrome does.
- A test checks that the extension id in `install.py` matches the public key in `extension/wxt.config.ts`.
- Every test database is created under pytest's temporary folder (`service/tests/conftest.py`).

## The contract between extension and service

The extension's tests turn the made-up, site-shaped pages in `extension/tests/fixtures/` into the contract files
in `service/tests/fixtures/` (as Vitest file snapshots), and the Python tests read those same files. A change on
either side of the contract fails a test. The site-shaped pages deliberately contain a seller name, phone and
address, so the tests also prove those are dropped. When the contract changes on purpose, update the snapshots
with `pnpm exec vitest run -u` in `extension/`, then change `service/src/carwatch/models.py` to match.

If the site changes its data, the popup shows "Sajt je promenio format" and nothing is stored. The fix starts in
`extension/lib/extract.ts`; the service side changes only if the contract does.

## Conventions

- Python: ruff with line length 120 and rules E, F, I, B, UP, SIM, RUF; mypy strict.
- No seller data, phone numbers or e-mail addresses in fixtures except the made-up ones the tests check are
  dropped.
- Messages shown to the user are in Serbian; code, comments and documents are in English.

## Documents

A change updates the documents it makes wrong in the same commit: [the architecture](../architecture/overview.md),
[the diagrams](../diagrams/), and the pages under [reference](../reference/) (the schema, settings, messages,
routes and commands are all listed there).
