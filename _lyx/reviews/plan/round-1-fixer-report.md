# Round 1 fixer report — plan

All 13 findings were worked in the plan directory only, with no git commands and the overview's `approved:` key left at `false`.
After the edits `lyx loom validate-plan` reports `ok: true` with informational `create-new-unit` findings only.

## Changes

- **F5 (card split).** Card 5 now holds D4 and D9's engine half: the hold in `Wait`, `AwaitOperator` retirement with its caller edits, and the `AskingOffset` doc.
  The new card 6 `06-hold-notice.md` holds D5's engine half: `Runner.SetNotifier`, `hold.go`, `RunState.StrandName` and `NotifiedOffset`, and the notice-only `hold_test.go`.
  Old cards 6–14 became 7–15 (files renamed, headings, Card Index, `Applies to` lines and every in-card cross-reference updated): attach 7, wiring 8, shed 9, webster 10, treadle 11, retire 12, stencils 13, corpus 14, live 15.
  `removal-order` now reads cards 5, 9, 10, 11, 12; `notice-shape` applies to cards 6, 13, 14; `legacy-asking-value` to cards 5, 7, 12.
  Card 7 lost its `rundir.go` target, since the `AskingOffset` doc moved to card 5.
- **F1.** Card 14 (was 13) adds `fixture_test.go`, `wait_test.go` and `startup_test.go` to its Edit targets, names the rename in its intent, and its `ImpactSummary` states the real blast radius.
- **F2, F3.** Card 11 (was 10) adds `result.go` (drop `RoundSummary.TriagePath` and its two fill sites in `run.go`), `profile.go` (comment) and `.gitattributes` (the stencil's line) to its targets, and the `engine_test.go` edit deletes the `Rounds[0].TriagePath` assertion.
- **F4.** Card 12 (was 11) adds nine files to its Edit targets (`burlerengine/engine.go`, `doc.go`, `smoke_cluster_test.go`, `frictionengine/reflect.go`, `mergeresolve/mergeresolve.go`, `webstercli/run.go`, `webstercli/ownership.go`, `loomcli/smoke_attachprobe_test.go`, `shuttleengine/run_test.go`), and states the end condition for the word `asking` in place of the symbol-only grep.
- **F6.** Card 5 has the held turn end carry the events-file offset just past its line, computed once in `pollEventsTick` and kept by `recordWaiting`; card 6 notifies an expired-shell hold with that offset; card 7's end-to-end attach test gains the `running`-record row with an expired-shell waiting line.
- **F7.** Card 6 states that a failed `NotifiedOffset` write logs Warn and skips that one notice, adds no return site to `Wait`, and leaves the tripwire's audited counts unchanged.
- **F8.** Card 13 (was 12) places the driver statement in `## Notifying the parent`: the generic line and a verbatim `parent_notice` each end with the tail, and "add nothing to it" becomes "add nothing to it but that tail".
  This is the opposite resolution to the review's suggested fix (no tail on the parent notice).
  I took it because D8 says every message to a working agent ends with the tail and the driver messages only its parent, so the review's version would have left the driver statement with no message to apply to.
  `claims_test.go` pins it, and the notice's forbidden characters are named.
- **F9.** Card 5's rewritten `TestRun_Wait_AwaitOperator_AskingNonTerminal` and `TestPollEventsTick_ShellExpiry` carry the hold-then-Done, died, timeout and expired-shell rows; `hold_test.go` in card 6 covers the notice only.
- **F10.** Card 15 (was 14) drops the last scenario; card 12's rewritten question smoke test sets a recording notifier and asserts one call, and card 12 now lists `Runner.SetNotifier` under `Uses`.
  Card 15 no longer sets a notifier.
- **F11.** Card 14 states each case's reading and what it adds over `background_test.go` and `skillload_test.go`, names the real source for the two #411 cases (tool-use and task ids), the skill-load case (hub orch run `6c8e3b92…`, `promptOffset` 649) and the continued-in case (`97d818aa-…`, continued in `8e5a1d83-…`), makes the harness run the real `SkillLoadMessage` and `ClassifySkillLoad` when a case names skills, and adds the notifier and `SetNotifier` use.
- **F12.** Card 1 adds the fork-label assertion to its waiting row and the `BackgroundTask.Label` doc update.
- **F13.** Card 1 names the first fixture by tool-use id and task id with the corrected line numbers (281, 283, 293, 297, 302), and the second transcript's verified lines (95, 356, 360, 362).

## Deferred

- **F11, two sources.**
  The `fork-killed-api-error` case names no real source: a bounded search of the hub's events files found no `background_tasks` entry with a status other than `running`, so none can be named without inventing one.
  The `master-fork-running` case names the hub orch run `6c8e3b92…` as its fallback, since no Master run with a running fork turned up.
  Card 14 tells the implementer to record the source in `case.yaml` and to report, never synthesize, a case whose real run reads differently.
  Finding a real killed-fork transcript is a search of `~/.claude/projects` that this round could not bound.
