# iris — troubleshooting

Run `bash scripts/doctor.sh` first: it isolates which layer is broken
(iris binary / browser / smoke capture).

## `iris: command not found`

- Not installed, or `~/.local/bin` not on PATH.
  Fix: `bash scripts/install.sh`, then add `~/.local/bin` to PATH
  or use the absolute path.
- `iris --version` should print e.g. `iris 0.4.1`.

## No Chrome-family browser found / `error: no browser`

- iris auto-detects Chrome, Chromium, Edge, Brave in standard locations;
  otherwise set `CHROME=/path/to/chrome` or pass `--chrome /path/to/chrome`.
- Linux headless server: `bash scripts/install.sh` fetches Chrome for Testing
  into `~/tools/chrome-linux64` and the required system libs.

## `Running as root without --no-sandbox is not supported` (Linux containers)

Chrome refuses to start as root ([crbug.com/638180](https://crbug.com/638180)),
and iris has no flag to forward `--no-sandbox`. You will see:

```
Error: failed to launch Chrome (install Google Chrome or pass --chrome)
  Caused by: ... Running as root without --no-sandbox is not supported.
```

`scripts/doctor.sh`, `scripts/install.sh` and the MCP wrapper handle this
automatically: when running as root they generate a shim at
`~/tools/chrome-no-sandbox` that execs the real browser with `--no-sandbox`,
and point iris at the shim. Doctor prints the resolved path as `CHROME_PATH`.

- Doing it by hand: create the two-line shim yourself and pass
  `--chrome ~/tools/chrome-no-sandbox` (or `export CHROME=…`).
- Better, where possible: run as a non-root user and skip the shim entirely.
- To opt out of the shim: `IRIS_NO_SANDBOX_SHIM=0`.

## `error while loading shared libraries: libnss3.so …` (Linux)

Chrome for Testing needs system libs. Install:

```bash
sudo apt-get install -y libnss3 libnspr4 libatk1.0-0 libatk-bridge2.0-0 \
  libatspi2.0-0 libcups2 libxkbcommon0 libxdamage1 libasound2t64 fonts-liberation
# older Debian/Ubuntu: libasound2 instead of libasound2t64
```

## Capture is blank / fonts or images missing

- The page needs more settle time: `--wait 1000` or `--wait-for '#ready-id'`.
- Content behind a lazy-load section on a non-`--full` viewport: use `--full`,
  or capture with `--selector` after `--wait-for`.
- Very heavy page: raise `--timeout 60`.

## Full page comes back @1x

Expected behavior: pages taller than Chrome's ~16k px render limit fall back
from @2x to @1x automatically. The JSON `scale` field tells you which you got.
Capture sections instead if crispness matters more than completeness.

## Selector surprises

- `--selector` = FIRST match in document order. Scope with a more specific
  selector (`section#hero h2`) rather than expecting all matches.
- `--selector` + `--full` conflict (pick one mode). `--padding` requires
  `--selector`.
- Cross-origin iframe content is invisible to iris.

## Batch: one URL failed, the rest captured

Normal: failures print `✗` and still count — exit code is 1 if ANY URL failed.
With `--json`, the failed entry is JSON with a non-`ok` status. Check each
result instead of trusting the exit code alone.

## MCP: `capture` tool not appearing

- Register with `command: iris`, `args: ["mcp"]` (binary on PATH or absolute).
- Start a fresh agent task/session — some clients cache the tool list.
- Liveness check: pipe an `initialize` JSON-RPC line to `iris mcp` (see
  `references/mcp-setup.md`).
- Remember: the CLI smoke test does NOT prove the MCP path.

## Windows

- Chrome paths are auto-handled; `--chrome` accepts the Chrome/Edge install
  path if auto-detection fails.
- `install.sh` targets Linux/macOS; on Windows use `cargo install
  iris-screenshot` or the prebuilt binary from the GitHub releases.
