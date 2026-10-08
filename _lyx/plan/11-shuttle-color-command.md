# Card 11 — Shuttle carries the segment and types the provider's color command

**Edit:**
- `internal/shuttleengine/spec.go`
- `internal/shuttleengine/engine.go`
- `internal/shuttleengine/run.go`
- `internal/shuttleengine/session.go`
- `internal/shuttleengine/fakes_test.go`
- `internal/shuttleengine/run_test.go`
- `internal/shuttleengine/session_test.go`
- `internal/shuttleengine/doc.go`
- `internal/shuttleengine/claudeengine/doc.go`
- `internal/testkit/shuttlefake/shuttlefake.go`
- `PATTERN.md`

**Create:**
- `internal/shuttleengine/claudeengine/color.go`
- `internal/shuttleengine/claudeengine/color_test.go`

**Uses:**
- `internal/segmentcolor/segmentcolor.go`
- `internal/reedengine/state.go`

**Intent:**
Carry a run's segment to reed and type the provider's color command into the fresh session, per D14, so Claude Code's prompt bar matches the strand's tmux tint.

`Spec` gains `Segment segmentcolor.Segment`, the loom segment the spawning module names for its role, and `ColorByCaller bool`, set only by a caller that types the color itself (the orch, card 14), which makes shuttle type none.
`validate` inspects neither.
`Runner.start` passes `string(spec.Segment)` as `reedengine.AddSpec.Segment`; reed resolves and returns the palette color on the added strand (card 3).

`Engine` gains `ColorSequence(color segmentcolor.Color) []PaneInput`: the key choreography that sets the session's display color, or no inputs for a provider without one.
After `awaitStartup` confirms the provider ready and before the skill load, `start` plays `ColorSequence(strand.Color)` into the strand's pane when the strand has a color and `ColorByCaller` is false.
An empty sequence types nothing; a failed play is logged via `logger.Warn` and never fails the launch, since the color is display only, like the pane title.
`Runner.TypeColor(guid string, color segmentcolor.Color) error` plays the same sequence into a live shuttle strand for the orch watcher, with the same checks `ReloadPlugins` makes (a shuttle strand, a live pane).

`claudeengine/color.go`: `(*Claude).ColorSequence` maps the palette color to Claude Code's `/color` name, the only place that mapping lives (`PATTERN-shuttle-provider-seam`): the eight palette names map to the same Claude Code names (`blue`, `purple`, `cyan`, `green`, `orange`, `yellow`, `pink`, `red`), and the sequence is `/color <name>` typed and submitted the way `ModelSwitchSequence` submits `/model`; a color outside the palette answers no inputs.
`shuttlefake.Engine` gains a `ColorSequenceFn` answering no inputs when nil, and the package-local fake in `fakes_test.go` follows.

PATTERN: a new one-line entry `PATTERN-spawn-color` under "Agents and prompts": coloring a spawned session — its spawning module names a segment on the launch spec, reed tints its bar button and border, and lyx types the provider's color command; no stencil asks an agent to set a color.

Tests:
`color_test.go` is one table over every palette color plus an unknown one, asserting the typed inputs.
`run_test.go` gains rows: a spec with a segment passes it on the add spec; a colored strand gets the color inputs after startup and before the skill-load message; no color, `ColorByCaller`, and an engine answering no inputs each type nothing; a failing color play still starts the run.
`session_test.go` gains a `TypeColor` row against a live strand and one against an unknown guid.

Docs: `shuttleengine` `doc.go` and `claudeengine` `doc.go` describe the color step, the capability and where the mapping lives.

**ImpactSummary:** the `Engine` interface gains one method (three implementers updated); `Runner.start` gains one non-fatal typing step; `Spec` gains two fields every caller may leave zero.
