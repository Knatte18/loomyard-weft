# Card 2 — Reed config: one color key per segment

**Edit:**
- `cmd/lyx/testdata/test-budget.yaml`
- `internal/reedengine/config.go`
- `internal/reedengine/template_posix.yaml`
- `internal/reedengine/template_windows.yaml`
- `internal/reedengine/template.go`
- `internal/reedengine/lifecycle.go`
- `internal/reedengine/config_test.go`
- `internal/reedengine/lifecycle_test.go`
- `internal/reedengine/doc.go`

**Create:**
- `internal/reedengine/segmentcolor.go`
- `internal/reedengine/segmentcolor_test.go`

**Uses:**
- `internal/segmentcolor/segmentcolor.go`

**Intent:**
Make each segment's color settable in reed's config (D13), with D13's table as the template defaults, and give reed its one resolver from a segment to a palette color.

Config: `Config` gains `SegmentColors map[string]string` under the YAML key `segment_colors`.
Both templates gain the block, one key per `segmentcolor.Segments()` value, with a comment naming the palette and stating that a value outside it fails `lyx reed up` and is no color everywhere else:
`coordinator: blue`, `discussion: purple`, `plan: cyan`, `webster: green`, `review: orange`, `describe: yellow`, `landing: pink`.
`configengine.LoadOrTemplate` already fills a present file's missing template keys from the template in memory, so a `reed.yaml` without the block, or with some keys missing, loads with the defaults (`PATTERN-config-strictness`).
`LoadConfig` validates nothing, since every `lyx reed` verb, pair teardown and the watchdog load it.
`template.go`'s doc comment names the new block among the plain literals.

`segmentcolor.go` (reedengine) holds:

- `validateSegmentColors(colors map[string]string) error`: every value must pass `segmentcolor.ParseColor`; the error names the key, the value and the palette, as the way forward (`set segment_colors.<key> to one of …`).
  Keys are checked in sorted order so the message is deterministic.
- `(e *Engine) segmentColor(segment string) (segmentcolor.Color, bool)`: empty segment → no color, silently; a segment with no key in `e.cfg.SegmentColors` → no color, logged via `logger.Warn` naming the segment; a value outside the palette → no color, logged naming key and value.
  This is the one place reed reads a segment's color, and only palette names leave it, so no config value carries format syntax into a tmux format or a typed command.

`ensureServerAndSessionLocked` calls `validateSegmentColors(e.cfg.SegmentColors)` in its pre-tmux validation block beside `mouse` and `watchdog`, so a typo fails `lyx reed up` before anything spawns, while `down`, `status`, teardown and an attach to a live session never reach it.
This is the only refusal; every other reader goes through `segmentColor`.

Tests:
`segmentcolor_test.go` (reedengine) is one table over `segmentColor`: a configured segment, an empty segment, a segment with no key and a non-palette value, asserting the color and ok.
`validateSegmentColors` is tested only through `Up`, its one caller: the boot-validation row below covers the refusal, and every existing `Up` test booting with the template defaults covers the all-valid map.
`config_test.go` gains rows: the embedded template decodes to D13's seven defaults, and a `reed.yaml` lacking `segment_colors` (and one lacking a single key) loads with the template defaults.
`lifecycle_test.go`'s `TestUp_BootValidation` table gains a row with `segment_colors.review: crimson` that fails `Up` naming the key, the value and the palette, before any tmux call.
`test-budget.yaml`'s `internal/reedengine` row is set to the package's top-level test count after this card.

`doc.go` gains a short paragraph: segment colors are read from config, resolved only through `segmentColor`, refused only at boot.

**ImpactSummary:** reed config gains one block and one boot check; every other reed reader is unchanged.
