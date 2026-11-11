# Setup

## Prerequisites

- Linux with Google Chrome or Chromium. The host is registered on Linux only.
- [uv](https://docs.astral.sh/uv/) and Python 3.13 or newer (`service/.python-version` pins 3.14 for uv).
- Node and [pnpm](https://pnpm.io/) for the extension.
- Optional: `notify-send` for desktop notifications on price drops.

## The host

```sh
cd service
uv sync
uv run carwatch install      # registers the host with Chrome and Chromium
uv run carwatch info         # where the database and the registration live
```

`carwatch install` writes the host manifest into `~/.config/google-chrome/NativeMessagingHosts/` (under
`$XDG_CONFIG_HOME` when it is set), and into the Chromium one if Chromium has a profile. The file points at
`service/.venv/bin/carwatch-host`, so run `install` again if you move the repository. `uv run carwatch uninstall`
removes it. Details: [commands](../reference/cli.md).

## The extension

```sh
cd extension
pnpm install
pnpm build                   # output: extension/.output/chrome-mv3
```

Then in Chrome: `chrome://extensions` → Developer mode → Load unpacked → `extension/.output/chrome-mv3`. The
manifest pins the extension id to `anfomkmnpkpjpjdlpngmmkedbplleiam`, the only id the host accepts.

## Check that it works

Open any search or ad on polovniautomobili.com. The popup shows whether captures arrive. If it says "Car Watch nije
instaliran", run `uv run carwatch install` in `service/`.

## Configuration

All settings are optional `CARWATCH_*` environment variables; the list is in
[configuration](../reference/configuration.md). Chrome starts the host, so the host sees Chrome's environment, not
your shell's.

## Run the UI

```sh
cd service
uv run carwatch ui           # http://127.0.0.1:8050/ until you stop it
```
