# iris — complete flag reference

Source of truth: `iris --help` and https://github.com/brijr/iris (re-check when
the skill may be stale — the tool iterates fast).

## Flags

```
-o, --out <PATH>      Output file (single URL) or directory (batch)
                      [default: ./<host>-<path>.png]
-s, --size <SIZE>     Viewport: WxH, or a preset:
                        desktop  → 1440x900 @2x
                        iphone   → 390x844 @3x (+ mobile user agent)
                        ipad     → 1024x1366 @2x
                      [default: desktop]
    --full            Capture the full page height
    --selector <CSS>  Capture the first element matching a CSS selector
    --padding <PX>    Uniform CSS-px padding around a selected element
                      (requires --selector; conflicts with --full)
    --dark            Emulate prefers-color-scheme: dark
    --format <FMT>    png | jpg | jpeg | webp  (a recognized --out extension wins)
    --wait <MS>       Extra settle delay (ms) after smart waiting
    --wait-for <CSS>  Wait until a selector exists before capturing
    --scale <N>       Device scale factor (overrides the preset's)
    --jobs <N>        Concurrent captures (default: min(4, number of URLs))
    --timeout <SECS>  Per-page budget (default: 30)
    --chrome <PATH>   Browser binary (auto-detected; env: CHROME)
    --json            Emit one JSON object per completed capture to stdout
```

Batch input: `iris -o shots/ url1 url2 …` or `cat urls.txt | iris - -o shots/`
(newline-separated URLs, `#` comments allowed).

## JSON output (`--json`)

One object per completed capture on stdout, in concurrent completion order.
Failures are JSON too; exit code 1 if anything failed.

```json
{"status":"ok","url":"https://example.com/","output":"/abs/path/example.com.png",
 "mode":"element","selector":"h1","padding":24,"css_width":180,"css_height":72,
 "scale":2.0,"format":"png","bytes":14231}
```

Fields to check programmatically: `status`, `scale` (1.0 = page hit the ~16k px
limit and fell back from @2x), `output` (absolute path), `bytes`.

## Tuning tips

- Slow/heavy page: raise `--timeout`, add `--wait 500` (extra settle after
  smart waiting), or `--wait-for '#id-when-ready'`.
- Long marketing page: `--full` scrolls to trigger lazy-load, then captures
  full height. Above ~8k CSS px expect the @1x fallback (reported in JSON).
- Retina is the default; pass `--scale 1` only to halve file size deliberately.
- Batch speed: default `--jobs` = min(4, URLs); raise `--jobs` on beefy CI.
- WebP/JPEG encode at quality 90; PNG is lossless and the default.
- One browser process per CLI invocation (concurrent tabs inside); the MCP
  server keeps one Chrome alive for its whole lifetime (faster repeated calls).

## What iris deliberately does NOT do

- Capture every match of a selector (first match in document order only).
- Capture cross-origin iframe contents.
- Interact with the page: no clicks, typing, scripted scroll, hover states.
- Diff or review workflows. (For those, pair with a browser automation tool.)
