# Card 1 — Claude reads a lone `%` in its input box as an empty box

**Create:**
- `plan:internal/shuttleengine/claudeengine#shellEOLMark` -> `const shellEOLMark = "%"`
- `plan:internal/shuttleengine/claudeengine#startClock` -> `type startClock struct`
- `plan:internal/shuttleengine/claudeengine#startClock.Now` -> `func (c *startClock) Now() time.Time`
- `plan:internal/shuttleengine/claudeengine#startClock.Sleep` -> `func (c *startClock) Sleep(d time.Duration)`
- `plan:internal/shuttleengine/claudeengine#TestStart_LoneEOLMarkBoxTakesTheStartupSends` -> `func TestStart_LoneEOLMarkBoxTakesTheStartupSends(t *testing.T)`

**Edit:**
- `internal/shuttleengine/claudeengine#Claude.IdleSession`
- `internal/shuttleengine/claudeengine#Claude.InputBoxText`
- `internal/shuttleengine/claudeengine#TestIdleSession`
- `internal/shuttleengine/claudeengine#TestInputBoxText`
- `internal/shuttleengine/claudeengine/doc.go`
- `pattern/PATTERN-pane-text-injection.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/shuttleengine/claudeengine#inputBoxInterior`
- `internal/shuttleengine/claudeengine#isBlankBoxInterior`
- `internal/testkit/shuttlefake#Reed`
- `internal/shuttleengine#Runner.StartGated`

**Intent:**
Decisions D1 and D2.
A ready Claude pane can show zsh's end-of-line mark, a lone `%`, on the row Claude later draws its input box on.
Claude never received that `%` as input, but the box reading counts it as a draft, so the pre-write guard holds every startup send until the start fails.

In `internal/shuttleengine/claudeengine/session.go`, declare `shellEOLMark`, the shell's end-of-line mark, with a comment naming zsh's `PROMPT_EOL_MARK` and why the mark can sit in a box Claude never got it as input for.
A box whose only non-blank content, once each method's own strip has run, is the single string `shellEOLMark` counts as blank, whether the `%` sits on the caret line or on a lower interior line.
- `IdleSession` reports idle for such a box.
  It keeps the running-turn check, the box location and its `isBlankBoxInterior` strip of `boxInteriorChars` unchanged.
  The only addition is that an interior whose non-blank lines, after that strip, amount to exactly `shellEOLMark` counts like an all-blank interior.
- `InputBoxText` keeps its strip of the caret and `│`, and returns `("", true)` when the joined text is exactly `shellEOLMark`.
- Any other content stays a draft: `%` beside other text (`% ls`, `50%`), `%%`, or another single character such as `#` or `x`.
Update both doc comments to state the reading.

Bound: this lets through exactly one thing, a box holding the character `%` alone.
Every caller of the reading sees the change: shuttle's pre-write guard, the startup sends and the orch watcher.
The operator-activity hold still runs before every guarded write.
The session idle probe's fallback for a strand with no run reads the box with no activity hold, so there a lone `%` reads idle with no operator check.
An operator draft of exactly `%`, typed outside `operator_quiet_s`, now reads as empty, and a paste can merge with it.
The knowledge stays in `claudeengine` (`PATTERN-shuttle-provider-seam`): `shuttleengine` gains no Claude specifics.

Tests:
- `TestIdleSession` gains these rows, each built from the `rule` box shape the table already uses:
  - a box holding `❯ %` reads idle;
  - a box whose caret line is blank and whose next interior line holds `%` reads idle;
  - a box holding `❯ % ls` reads not idle;
  - a box holding `❯ %%` reads not idle;
  - a box holding `❯ #` reads not idle.
  The `%%` row catches an implementation that strips every `%` rather than matching the lone mark, which the `% ls` row cannot catch.
- `TestInputBoxText` gains these rows:
  - `❯ %` reads `("", true)`;
  - `❯ % ls` reads `("% ls", true)`.
- New file `internal/shuttleengine/claudeengine/start_test.go`, in `package claudeengine` (the internal test package, since `claudeengine` already imports `shuttleengine` and `internal/testkit/shuttlefake` imports neither back).
  It holds `startClock`, a fake clock whose `Sleep` advances `Now` with no real wait.
  It also holds `TestStart_LoneEOLMarkBoxTakesTheStartupSends`:
  - Build `New()` (the real Claude engine) and a `shuttlefake.Reed`.
  - The fake reed's `AddStrandFn` returns a strand with the GUID `strand-1` and a palette color, so start types the color.
    `AddStrandFn` skips the fake's own strand registration, so the test seeds `Strands` with `{GUID: "strand-1", PaneID: "%1", Live: true}` before the start, and `Status` lists the strand live.
  - Its `CapturePaneFn` always returns `IdleInputBoxFixture` with the caret line `❯ %`, which `Startup` reads as ready and every box reading sees.
  - The test sets `HOME` to a temp dir with `t.Setenv`, so Claude's `ProcessLiveness`, which the send guard's session reading consults, reads an empty session registry rather than the developer's.
    That makes it process-global, so the test does not call `t.Parallel` and says why.
  - The config is the shipped template from `shuttleengine.LoadConfig(t.TempDir(), "shuttle")`, the way `ttlConfig` loads it.
  - Build the runner with `shuttleengine.NewRunner` over a temp worktree, set its clock with `SetClock(&startClock{...})`, and call `StartGated` on a spec that names one skill, with a short `SkillLoadTimeout` so the load turn's wait for a turn end ends quickly on the fake clock.
  - Assert that `StartGated` returns a run and a nil error.
  - Assert that `PasteSubmitCalls` holds, in order, the color command `ColorSequence` gives for that color, the `SkillLoadMessage` for the skill, and the launch's `PromptLine`.
  - Assert that the log, captured with `logcapture`, has no `could not type the segment color` line.
  Before this card the guard reads the `%` box as busy until `send_ready_timeout_s`, so the color send logs that warning, the skill-load send fails with `ErrSessionBusy` and `StartGated` returns an error wrapping `ErrNotStarted`.
  This is the one start-level test in `claudeengine`.
  No existing test runs a start against Claude's own box reading: the corpus replay drives a `shuttlefake.Engine` that reads no box.
- `cmd/lyx/testdata/test-budget.yaml`: raise the `internal/shuttleengine/claudeengine` row from 103 to 104 for the new top-level test.

`internal/shuttleengine/claudeengine/doc.go`: in the paragraph on how the engine takes text, add one sentence saying that the box reading counts a box holding only the shell's end-of-line mark `%` as empty.

`pattern/PATTERN-pane-text-injection.md`: under the pre-write guard bullet, add one line, one sentence per line.
It says that the provider's box reading defines "empty", and that Claude's counts a lone `%`, the shell's end-of-line mark left on screen, as empty.
It also says what that lets through: an operator draft of exactly a lone `%`, typed outside `operator_quiet_s`, reads as empty, and a write can merge with it.

**ImpactSummary:** Two Claude box-reading methods widen by one input, a box holding `%` alone, and every caller of them sees it: shuttle's pre-write guard and send confirmation, the startup sends and the orch watcher. No signature changes.

**Commit:** 1: Claude reads a lone `%` in its input box as an empty box
