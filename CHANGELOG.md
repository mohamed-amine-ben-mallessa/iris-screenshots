# Changelog

All notable changes to the iris-screenshots marketplace and plugin.

## [0.2.1] — 2026-09-16

### Fixed
- **Root containers could never capture anything.** Chrome refuses to start as
  root without `--no-sandbox` (crbug.com/638180) and iris has no flag to
  forward it. `doctor.sh`, `install.sh` and the MCP wrapper now generate a
  `~/tools/chrome-no-sandbox` shim when running as root and point iris at it
  (`IRIS_NO_SANDBOX_SHIM=0` opts out). Documented in `troubleshooting.md`.
- **Skills referenced `${CLAUDE_SKILL_DIR}`**, which Claude Code does not
  expand — every preflight resolved to `/scripts/doctor.sh` and failed. All
  three skills now use paths relative to the skill directory.
- **`allowed-tools` used the wrong syntax** (`Bash(iris *)`); prefix rules are
  `Bash(iris:*)`, and the `doctor.sh`/`install.sh` patterns did not account for
  the `bash ` prefix of the documented command, so they never matched.
- **Shell scripts were not executable** (mode 0644). `.mcp.json` invokes
  `scripts/iris-mcp.sh` directly, so the bundled MCP server failed to start.
- **`install.sh` aborted as root**: `SUDO="$(id -u)"` resolved to `0`, so the
  script ran `0 apt-get install …` (`0: not found`) and `set -eu` killed the
  install — precisely the container case the installer targets.
- `plugin.json` had no `author` and `marketplace.json` no `description`, so
  `claude plugin validate --strict` failed and CI was red on the first push.

### Changed
- Chrome for Testing is resolved from `last-known-good-versions.json` instead of
  a hardcoded build, with the pinned version as a fallback.
- The MCP wrapper now fails loudly (`die`) when a bootstrap step fails, instead
  of exec'ing a browser that was never installed.
- CI gained real teeth: the shell-syntax step used `find -exec sh -n` which
  returns 0 even on a syntax error. It now fails, and new steps check the
  marketplace manifest strictly, the executable bits, that the three copies of
  `doctor.sh`/`install.sh` stay in sync, and that `CLAUDE_SKILL_DIR` never
  comes back.

## [0.2.0] — 2026-09-16

### Added
- **Bundled MCP server** (`plugins/iris/.mcp.json` + `scripts/iris-mcp.sh`): the
  `capture` tool is available as soon as the plugin is enabled. The wrapper is
  self-bootstrapping — it installs `iris` + Chrome for Testing on first start
  and keeps the stdio JSON-RPC stream clean (bootstrap noise goes to a log).
- **`responsive-audit` skill** (`/responsive-audit`): captures desktop, iPhone
  and iPad (+ dark on request) with identical settings, then produces a
  structured audit (overflow, overlap, layout, typography, contrast) with
  severity-graded findings.
- **`visual-diff` skill** (`/visual-diff`): compares two states (A/B URLs or
  before/after) with a cheap JSON pre-check, then a structured Added / Removed
  / Moved / Restyled / Reflow report and an explicit verdict.
- **`visual-reviewer` subagent** (`@visual-reviewer`): proactive visual QA —
  screenshots the running app after UI changes and reports concrete defects.
- **CI**: `.github/workflows/validate.yml` runs `claude plugin validate
  --strict`, shell syntax checks and JSON manifest checks on push/PR.

## [0.1.0] — 2026-09-16

### Added
- `screenshot` skill (`/screenshot`): iris-powered live-website screenshots —
  full page, CSS element, desktop/iPhone/iPad presets, dark mode, concurrent
  batch, JSON output. Self-installing (`doctor.sh` + `install.sh`).
- References: full flag reference, MCP setup guide, troubleshooting.
- Marketplace + plugin manifests, viral README (hero, flow diagram, MCP dark
  banner, real full-page demo, mascot GIF).
