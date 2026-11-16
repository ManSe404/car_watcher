# 2. Native messaging instead of a local HTTP service

Date: 2026-11-06

## Status

Accepted. Introduced with commit `c83ed50` ("reach Car Watch through native messaging instead of an HTTP
service"). It replaced the local HTTP service of the first Car Watch version.

## Context

In the first version the extension called a local FastAPI service on port 8050 with a token baked into the
extension build. The service had to be running before anything could be collected, which meant starting it by
hand or installing a systemd unit.

## Decision

Chrome starts the Python host, `carwatch-host`, for every message the extension sends with
`runtime.sendNativeMessage` and takes its one reply over stdin and stdout. Nothing has to run beforehand: no
service to start, no port, no token in the build, no systemd unit.

- `carwatch install` registers the host with Chrome and Chromium (Linux) through a manifest that allows one
  extension id.
- The extension id is pinned by a public key in `extension/wxt.config.ts`; a test keeps it in step with
  `service/src/carwatch/install.py`.
- Work that outlives one message, the fair-price valuation, runs as a detached `carwatch revalue` under a lock
  file.
- Host errors worth retrying (a locked database) keep the capture in the extension's outbox.
- The web UI stays, as `carwatch ui`, started only when wanted.

## Consequences

- Every message starts a new Python process, so the host must answer fast; scikit-learn is imported only by the
  valuation.
- Chrome refuses replies over 1 MB; the host turns an oversized reply into an error.
- The host sees Chrome's environment, not the shell's, so `CARWATCH_*` variables reach it only when Chrome was
  started with them.
- The manifest names the host program by its path in the project's virtual environment: moving the repository
  needs `carwatch install` again.
- Registration exists for Linux only; macOS and Windows need their own.
- Several host processes can run at once and share the SQLite file through WAL mode and a busy timeout.
