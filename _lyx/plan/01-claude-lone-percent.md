# Card 1 — claude-lone-percent

**Create:**
- `plan:internal/shuttleengine/claudeengine#boxHoldsOnlyPercent` -> `func boxHoldsOnlyPercent(interior []string) bool`
- `internal/shuttleengine/claudeengine/lonepercent_start_test.go`

**Edit:**
- `internal/shuttleengine/claudeengine#Claude.IdleSession`
- `internal/shuttleengine/claudeengine#Claude.InputBoxText`
- `internal/shuttleengine/claudeengine#TestIdleSession`
- `internal/shuttleengine/claudeengine#TestInputBoxText`
- `pattern/PATTERN-pane-text-injection.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:** Claude's input-box reading treats a box whose only content is a lone `%` as empty (D1, issue #549), and the invariant's background says so (D2).
`boxHoldsOnlyPercent(interior)` is true when exactly one interior line is non-blank under `isBlankBoxInterior` and that line, with the `boxInteriorChars` and whitespace removed, is the single character `%`.
`IdleSession` then answers true for such a box, and `InputBoxText` answers `("", true)` for it, through that one helper so the two readings never disagree.
Any other content stays a draft: `%` beside other text on the same or another line, two `%` lines, and any other single character such as `#`.
`IdleSession`'s doc comment, which says any non-empty box classifies as not idle, names the `%` exception.
The `%` knowledge stays in `claudeengine` (`PATTERN-shuttle-provider-seam`); `shuttleengine` gains no Claude specifics.
What it lets through: a real operator draft of exactly `%`, typed outside `operator_quiet_s`, reads as empty and a write can merge with it.
The bound is the one character `%` alone in the box; the operator-activity hold still runs before every write behind the pre-write guard, and only the session idle probe's fallback for a strand with no run record reads the box with no activity hold.
`pattern/PATTERN-pane-text-injection.md` gains one line under the pre-write guard bullet: the provider's box reading defines "empty", Claude's reads a lone `%`, the shell's end-of-line mark left on screen, as empty, and an operator draft of exactly a lone `%` typed outside `operator_quiet_s` reads as empty and a write can merge with it.
The classifier rows, on captures built as rule, caret line, rule, `? for shortcuts` footer: `TestIdleSession` gains `❯ %` (idle), `│ ❯ %     │` between boxed side bars (idle), `❯ % half-typed` (not idle), `❯ #` (not idle), `❯ %` over a second line `more` (not idle) and `❯ %` over a second line `%` (not idle).
`TestInputBoxText` gains `❯ %` (`""`, true), `❯ % half-typed` (`"% half-typed"`, true), `❯ #` (`"#"`, true) and the `%`-over-`more` capture (`"% more"`, true).
The new test file holds one top-level test in package `claudeengine_test`, `TestStart_LonePercentBoxLetsTheStartupSendsThrough`, which runs `Runner.StartGated` with a small wrapper type embedding `*claudeengine.Claude` that overrides only `Prepare` to return a canned `shuttleengine.Launch` whose `PromptLine` is a short sentence, so every reading and the color sequence are Claude's own.
It runs on `shuttlefake.Reed` with `AddStrandFn` returning a live strand whose `Color` is `segmentcolor.Blue`, `StatusFn` reporting it live, `CapturePaneFn` returning a ready bottom box (rule, caret line, rule, `claudeengine.ReadyFooterFixture`), and a clock type defined in the file whose `Sleep` advances `Now`, set through `Runner.SetClock`, so no wall time passes.
The runner is built with `shuttleengine.Config{PollIntervalMS: 1000, LivenessEveryNPolls: 1, RunTimeoutMin: 60, StartupTimeoutS: 30, BackgroundShellWaitMin: 10}`, the shape `corpus_test.go` uses, and the spec sets `Prompt`, an absolute `OutputFiles` entry under `t.TempDir()`, `Role` and `Timeout`.
Row one has the caret line `❯ %`: `StartGated` returns a run, and the reed records a `PasteSubmit` whose text is `/color blue` and one whose text is the prompt line, so the color send and the deferred prompt delivery both pass the pre-write guard and land because the box reads empty.
Row two has the caret line `❯ % half-typed`: the reed records no `PasteSubmit` at all and `StartGated` returns an error wrapping `ErrNotStarted` and `ErrSessionBusy`, since the color send is held and only logged while the prompt delivery is held and tears the start down, so a draft containing `%` is still protected.
The skill-load send is not driven by a row: it takes the same `sendVerified` call as the prompt delivery (`loadSkillTurn`), a skill turn needs a provider events file this fake cannot honestly produce, and the row above already shows that call passing the guard on a lone `%`.
`cmd/lyx/testdata/test-budget.yaml` raises the `internal/shuttleengine/claudeengine` row by one for that test.

**ImpactSummary:** The reading is consumed by shuttle's pre-write guard, the startup sends and the orch watcher's idle probe, all through `IdleSession` and `InputBoxText`; only a box that is exactly a lone `%` changes class, and every other fixture row stays as it was.

**Commit:** `1: claude-lone-percent`
