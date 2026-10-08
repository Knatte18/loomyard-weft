# Card 3 — Strand label, color and window marker as tmux user options

**Edit:**
- `cmd/lyx/testdata/test-budget.yaml`
- `internal/reedengine/state.go`
- `internal/reedengine/strand.go`
- `internal/reedengine/spawn.go`
- `internal/reedengine/attach.go`
- `internal/reedengine/lifecycle.go`
- `internal/reedengine/spawn_test.go`
- `internal/reedengine/attach_test.go`
- `internal/reedengine/strand_test.go`
- `internal/reedengine/lifecycle_test.go`
- `internal/reedengine/doc.go`

**Create:**
- `internal/reedengine/paneoptions.go`

**Uses:**
- `internal/reedengine/segmentcolor.go`
- `internal/agentname/agentname.go`

**Intent:**
Give every strand pane the two pane user options of D2 and the strand window the marker of D3, so the bar (card 5) and the bindings (card 7) can find strands without reading the current window.

Record: `AddSpec` gains `Segment string`, which `addStrandLocked` stamps onto the new record; `Strand` gains `Segment string` (`json:"segment,omitempty"`), an opaque display field reed never branches on except to look up its color, so a strand recorded before this change decodes with no segment.
`Strand` also gains `Color segmentcolor.Color` tagged `json:"-"`: never persisted, filled with the resolved palette color (card 2's `segmentColor`) on the strand that `AddStrand`, `AddStrandUnless` and `ReplaceStrand` return, so shuttle (card 11) learns the color from the add.
`StrandStatus` gains `Color segmentcolor.Color`, filled the same way in `Status`.

`paneoptions.go` holds:

- `strandLabel(name string) string`: the role segment of `agentname.Parse(name)`, or the full name when it does not parse (`PATTERN-agent-name`: nothing else splits a name).
- `(e *Engine) setStrandPaneOptionsLocked(paneID string, s Strand)`: `set-option -p -t <pane> @strand <label>`, then `@strand_color <tmux color>` when `segmentColor(s.Segment)` resolves (mapped through `segmentcolor.TmuxColor`), else `set-option -p -u -t <pane> @strand_color`.
- `(e *Engine) reassertStrandPaneOptionsLocked(st *ReedState, live []LivePane)`: calls the above for every strand bound to an alive pane in live.
- `(e *Engine) markStrandWindowLocked(windowTarget string)`: `set-option -w -t <window> @lyx_strands 1`.

Every call is display only: a failure is logged via `logger.Warn` and never fails the step, like the pane title beside it.
Selvage never gets either pane option; their absence is what keeps it off the bar.

Call sites:
`launchStrandLocked` (`spawn.go`) calls `setStrandPaneOptionsLocked` right after it sets the pane title, so add, update and resume (every strand-realizing path) set both options.
`upLocked` and `Resume` (`lifecycle.go`) call `markStrandWindowLocked` once Selvage exists and the strand window resolves, which is the boot.
`AttachArgv`'s pre-flight, right after it resolves the strand window and before the `window-size` and `#{status}` readbacks, calls `markStrandWindowLocked`, lists the window's panes and calls `reassertStrandPaneOptionsLocked` with that listing, so a session or strand spawned by an older lyx gets them at the next attach whichever chain guard later suppresses the chain.
A failed listing is logged and skips the re-assert only.

Tests (fake tmux, untagged):
`spawn_test.go`'s `TestLaunchStrandLocked_SplitWindowAndPaneTitleCalls` gains launch fixtures for a colored strand, a strand with no segment and a strand whose segment has no config key, asserting both pane options (or the `@strand_color` unset) after the title; across them the strand names cover the four label shapes (a prime name, a task name, the numbered role `burler-review-2` and an unparseable name), so the `@strand` value pins `strandLabel` through the real launch path and the helper gets no test of its own.
`attach_test.go` asserts the pre-flight marks the window and re-asserts the options on a live bound pane, not on Selvage, also when the `window-size` or `#{status}` readback suppresses the chain.
`test-budget.yaml`'s `internal/reedengine` row is set to the package's top-level test count after this card.
`strand_test.go` asserts `Segment` persists and the returned strand carries `Color`; `lifecycle_test.go` asserts `Status` carries `Color` and that a recorded strand without a segment reports none.

`doc.go` documents `@strand`, `@strand_color` and `@lyx_strands`: who sets them, when, and that no Go decision reads them back.

**ImpactSummary:** reed's add/launch/attach/status paths gain non-fatal option writes and two record fields; callers of `AddSpec` compile unchanged.
