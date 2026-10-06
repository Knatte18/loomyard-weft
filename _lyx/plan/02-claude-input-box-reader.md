# Card 2 — claude-input-box-reader

**Create:**
- `plan:internal/shuttleengine#InputBoxReader` -> `type InputBoxReader interface`

**Edit:**
- `internal/shuttleengine/claudeengine/session.go`
- `internal/shuttleengine/claudeengine/session_test.go`

**Intent:** D1, provider half: reading the input box is a provider capability beside the existing idle probe, under `PATTERN-shuttle-provider-seam`.

Declare `InputBoxReader` in `internal/shuttleengine/engine.go`, beside `SessionCycler` and `SkillLoader`, as an optional capability an `Engine` may also implement, with two methods:

- `InputBoxText(capture string) (text string, ok bool)`: the text the provider's input box holds in capture, with the caret, side bars and line breaks stripped; ok is false when capture shows no input box the provider can read.
- `SubmitSettle() time.Duration`: how long after an Enter the box needs to be redrawn before it is read, so a send whose Enter landed is not read as still pending.

The doc comment says a provider without it keeps the appearance-only send check, and that the matching rule lives in shuttleengine, not here.

Implement both on `*Claude` in `session.go`, with a compile-time assertion beside the existing ones.
`InputBoxText` locates the box exactly as `Claude.IdleSession` does (the last caret line, walking up to the first top rule and down to the first rule), and returns the interior lines joined with the caret marker and side bars removed; no box found answers ok false.
Factor the box location out of `IdleSession` into one helper both methods call, so the two cannot disagree about where the box is.
`SubmitSettle` returns a provider-owned constant of a few hundred milliseconds (`submitRedrawSettle`), long enough for Claude Code to redraw the box after an Enter.

Tests in `session_test.go`, table-driven over capture fixtures (the existing `testdata/pane-idle-empty.txt` and `pane-idle-draft.txt` plus inline captures): an empty box answers ok true with empty text; a box holding a draft answers its text; a wrapped two-line draft answers both lines' text; a collapsed paste placeholder answers the placeholder text verbatim; a capture with no box (a permission prompt, `testdata/pane-permission-prompt.txt`) answers ok false.
`SubmitSettle` is pinned to the constant.

**ImpactSummary:** Adds a new optional shuttleengine capability and its Claude implementation; IdleSession keeps its behaviour through the shared box-location helper.
