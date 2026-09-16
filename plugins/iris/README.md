# Screenshot (iris) — Claude Code plugin

Give Claude Code a reliable camera for live websites. Powered by
[iris](https://github.com/brijr/iris) (Rust CLI + MCP, MIT): full page, single
CSS element, mobile/desktop presets, dark mode, concurrent batch, JSON output.

## What you get

- `/screenshot` — capture any page (full, element, mobile, dark, batch, JSON).
- `/responsive-audit` — desktop + iPhone + iPad (+ dark) side by side →
  severity-graded visual audit with concrete fixes.
- `/visual-diff` — A/B or before/after comparison → Added / Removed / Moved /
  Restyled / Reflow report with an explicit verdict.
- `@visual-reviewer` — subagent for proactive visual QA after UI changes.
- **Bundled MCP camera** — `.mcp.json` auto-registers the `iris` MCP server
  (`capture` tool, pixels inline). The wrapper (`scripts/iris-mcp.sh`) is
  self-bootstrapping: first start installs iris + Chrome when missing.
- **Self-installing skills**: each skill ships `doctor.sh` + `install.sh`, so
  any environment becomes operational with one command.

## Install

From the marketplace:

```
/plugin marketplace add mohamed-amine-ben-mallessa/iris-screenshots
/plugin install iris@iris-screenshots
```

Locally, point Claude Code at this directory:

```bash
claude --plugin-dir /path/to/iris-screenshots/plugins/iris
# or copy the skill for personal use:
cp -r skills/screenshot ~/.claude/skills/
```

Verify:

```bash
claude plugin validate /path/to/iris-screenshots/plugins/iris
```

## Requirements

- Network access for first-run install (iris binary + Chrome for Testing on
  machines without a browser).
- Linux: `sudo` for system libraries on first run (apt).
- Linux as root (containers, CI): Chrome will not start without
  `--no-sandbox`; the scripts generate a `~/tools/chrome-no-sandbox` shim
  automatically and point iris at it (`IRIS_NO_SANDBOX_SHIM=0` to opt out).
- macOS: any installed Chrome/Chromium/Edge/Brave.

## Structure

```
iris/
├── .claude-plugin/plugin.json      # manifest (name, version, metadata)
├── .mcp.json                       # bundled MCP camera (auto-registered)
├── scripts/iris-mcp.sh             # self-bootstrapping MCP wrapper
├── agents/visual-reviewer.md       # proactive visual QA subagent
└── skills/
    ├── screenshot/
    │   ├── SKILL.md                # entry point: preflight, command table, conventions
    │   ├── references/             # flags · mcp-setup · troubleshooting
    │   └── scripts/                # doctor.sh · install.sh
    ├── responsive-audit/           # SKILL.md + scripts/
    └── visual-diff/                # SKILL.md + scripts/
```

## License

MIT (plugin). Iris is MIT — https://github.com/brijr/iris
