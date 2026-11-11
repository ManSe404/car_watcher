# Commands

Two programs come with the service (`[project.scripts]` in `service/pyproject.toml`). Run them from `service/` with
`uv run`.

| Command | What it does |
|---|---|
| `carwatch install` | Writes the native-messaging host manifest for Google Chrome, and for Chromium when it has a profile folder (Chrome alone when neither has one). The manifest points at `carwatch-host` in the project's virtual environment and allows only the Car Watch extension id. Prints where it wrote and the extension id. |
| `carwatch uninstall` | Removes the manifests `install` wrote. |
| `carwatch info` | Prints the database path, the host program, each installed manifest and the extension id. |
| `carwatch ui` | Runs the web UI on `CARWATCH_HOST:CARWATCH_PORT` until you stop it. |
| `carwatch revalue` | Recomputes the fair prices now; says so and does nothing when another valuation holds the lock. |
| `carwatch-host` | The native-messaging host. Chrome starts it; it reads one message on stdin, answers on stdout and exits. Not meant to be run by hand. |

`install`, `uninstall` and `info` work on Linux only and exit with a message elsewhere. The manifest folders are
`google-chrome/NativeMessagingHosts/` and `chromium/NativeMessagingHosts/` under `$XDG_CONFIG_HOME`, or under
`~/.config` when it is not set: the per-user places Chrome and Chromium read on Linux. The manifest file is named
after `HOST_NAME` in `service/src/carwatch/install.py`, with `.json` added.
