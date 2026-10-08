# Plan-Review round 1 — fixer report

Nothing was fixed in the plan.
Every finding is left on purpose by the parent's call (`ly:orch`, received during this round):
cards 1-10 and `00-overview.md` stay exactly as Webster recorded them, and no card is added.
Cards 1-10 are already implemented on the task branch, so the round's gate checks them against a HEAD that already carries them (lyx issue #463).
Webster cannot take a follow-up card on this run's record.
The color-after-readiness decision will be fixed forward on the task branch at Publish instead.

Each finding was checked against the code and the decision record, and every premise holds.
The evidence below is for the Publish fix-forward.

## Changed

Nothing.

## Disputed

None.

## Deferred

- **F1 (BLOCKING, scope): no card carries the two `/color` decisions.**
  The premise holds.
  `grep color` over `_lyx/plan` finds nothing.
  The code still types the color raw: `internal/shuttleengine/run.go:494` calls `playInputs(r.reed, strand.GUID, r.engine.ColorSequence(strand.Color))` after `awaitStartup`, and `Runner.TypeColor` (`internal/shuttleengine/session.go:192`) calls `playInputs(... ColorSequence(color))`.
  Its only caller with `ColorByCaller` set is the orch watcher (`internal/orchcli/start.go:78`), and the watcher logs a failure without returning it (`internal/orchengine/watcher.go:308`).
  The loom driver colors through start's block, not through `TypeColor`.
  Notes for the fix-forward:
  - `sendVerified` takes text, but `ColorSequence` returns a `[]PaneInput` choreography (`claudeengine/color.go`).
    The provider seam therefore needs a text form of the color command (for example a `ColorCommand(color) string` on `Engine`), so the command text stays in `claudeengine` (`PATTERN-shuttle-provider-seam`).
  - A launch that carries its prompt on the launch line (a spec with no skills, `claudeengine.go:174`) is busy from its first moment.
    A color send through the idle wait would hold start for up to `send_ready_timeout_s` and then only log a Warn.
    A launch that defers its prompt (`launch.PromptLine != ""`) is idle with an empty box after `awaitStartup`, so the color send lands before the skill load.
    The loom driver is in this group (`driverSkills = {"scribe:prose"}`).
  - The tests to extend are `claudeengine/color_test.go`'s `TestColorSequence_TypesTheColorCommandForEveryPaletteColor`, `session_test.go`'s `TestRunner_TypeColor_PlaysTheEngineSequenceOnALiveShuttleStrand` and `testkit/shuttlefake`'s `ColorSequenceFn`.
    The three loomcli tmux-tier tests the decision record names show whether the fix works.
  Deferred by the parent's call.
- **F2 (BLOCKING, design): an empty box reads as landed for every `ErrSubmissionNotLanded` cause.**
  The premise holds against card 4, and the implementation does the same thing.
  In `internal/shuttleengine/send.go`, `failUnlanded` returns nil whenever `clearUnlandedText` reports an empty box.
  `sendVerified` (`run.go:1150-1153`) routes the never-appeared failure through `failUnlanded` too, and `settleAndConfirm` routes the unsettled failure the same way (`send.go:273-274`).
  So an input the TUI swallowed, or a typed text that vanished before any Enter, comes back as a successful send on Claude.
  D2, D4 and the acceptance criterion "A sent text that never appears in the pane fails with `ErrSubmissionNotLanded`" license the empty-box-means-landed inference only after an Enter.
  Deferred by the parent's call.
  This is a live correctness defect in the shipped code, not just a plan gap.
- **F3 (MEDIUM, design): the status-cap teardown rests on a false premise.**
  The premise holds.
  `awaitStartup` (`internal/shuttleengine/wait.go`) now routes all three status-cap exits through `abandonStartup(OutcomeDied, ...)`.
  But a spec without skills carries its prompt on the launch line, so its agent may already be working when reed's status fails, which contradicts card 5's Bound.
  `internal/loomcli/start.go:63-70` still records the opposite residual ("the strand is left in place only when shuttle's readiness check could not get an answer from reed at all ... tearing the strand down could kill a working driver"), so that comment is now stale.
  Deferred by the parent's call.
- **F4 (LOW, consistency): card 7's lock list omits `loadSkillsThenPrompt`'s save.**
  The premise holds against the card text.
  The code already takes `recordMu` around that save and calls `saveState` (`run.go:543-546`), so the gap is in the plan's wording only.
  Deferred by the parent's call.

## Gate findings (after the review fix)

After this report was first written, the round's gate flagged blocking findings on cards 2, 3, 4, 6, 7, 9 and 10.
None was repaired, by the parent's same call, which named these gate findings explicitly.
They come from cards 1-10 already being implemented, and the gate resolves the plan against a HEAD that already carries them (lyx issue #463):

- `create-already-exists` on every `Create` target of cards 2, 3, 4, 6, 7 and 9: the symbols exist because those cards ran.
- `glyph-not-found` and `path-missing` on card 4's `sendExtraEnters`, card 9's `shedfake#reedOps` and card 10's deleted removers (`burlerengine/remover.go`, `remover_test.go`, `shedfake#StrandRemover` and its method, `websterengine#removeStrandIfLive`): the symbols are gone because those `Delete` targets ran.
- `caller-uncovered` on card 3 (`sendVerified`, referenced at `send.go:438`), card 4 (`confirmSubmitted`, `send.go:271`) and card 9 (`RemoveRecoveryStrands`, `websterengine/stopper_test.go:41,52,61`): those references sit in `send.go` and `stopper_test.go`, files the implementation created, so no pre-implementation card lists them.

Repairing them would rewrite the recorded cards to describe HEAD instead of the work, which is what the parent ruled out.

The gate's second pass reported the same set unchanged (create-already-exists, glyph-not-found, path-missing and caller-uncovered on the same cards).
They stay unrepaired for the same reason, and the parent has been told that the gate keeps blocking on them.
The third pass reported the same set again and is handled the same way.
