# Round 1 fixer report

All four findings are fixed, one commit each, and nothing was pushed.
After the last commit `go build ./...`, `go vet` (no tag, `integration`, `tmux`, and `llm` over `shuttlecli` and `loomshed`) and the untagged `go test ./...` pass.
`go test -race -count=5` over the four touched held-run tests passes, and `go test -race ./internal/shuttleengine/` passes.

## Fixed

- **F4** (`e65e2d377`): deleted the `session-continued-in` corpus case directory.
  Its Stop line was identical to `issue411-prune-rest`'s, so it replayed the same reading.
  The real run holds no turn end after the `continued-in` entry, so the plan's case (Stop lines carrying the continuing session's transcript path) cannot be built from real data and was not synthesized.
  The parent should know that card 15's `session-continued-in` entry is therefore absent by design.
- **F3** (`67ac785f4`): reworded the comments that still named asking as a shuttle run outcome in `shuttleengine/spec.go` (three), `spec_test.go`, `run.go`, `claudeengine/settings.go` and `burlerengine/engine_test.go`.
- **F2** (`a5f96211f`): added `t.Parallel` to `TestWait_HeldTurnEndNotice`, `TestRun_Wait_HeldTurnEnd` and `TestDispositionCandidate_AskingReentry` and their subtests, and to the top level of `TestAttach_LegacyAskingAndHeldRuns`.
  Each builds its own fixture over `t.TempDir()`, fakes and a fake clock, and the race detector found no shared state.
- **F1** (`49a26f479`): reflowed the comments this diff added or changed to one sentence per line, with no wording change, in `background.go`, `hold.go`, `wait.go`, `rundir.go`, `attach.go`, `run.go`, both `doc.go` files, and the test files `attach_asking_test.go`, `wait_test.go`, `waiting_test.go`, `background_test.go`, `modelpin_test.go`, `planwrite_test.go`, `smoke_guardrail_test.go`, `smoke_interrupt_test.go` and `smoke_turns_test.go`.
  One wording change beyond a reflow: the `RunState.AskingOffset` doc still said Attach compares it against the events file's size, which card 7 removed, so it now says Attach replays from it and counts the ask before it as notified.
  In `attach.go`'s `leftoverThenAgeVerdict` doc I restored the original "timeout/asking" text instead of reflowing a sentence inside a legacy-wrapped paragraph.

## Deferred

Nothing is deferred.

One scope note on F1: where the diff only deleted or substituted a few words inside a paragraph that was already hard-wrapped on base (for example in `treadleengine/judge.go`, `judgeverdict.go`, `judge_test.go`, `state_test.go`, `template_test.go`, `burlerengine/engine.go`, `loomcli/driverspec.go` and the `smoke_guardrail_test.go` file header), I left the paragraph's wrap alone.
`PATTERN-comment-line-breaks` calls those lines legacy and forbids rewrapping untouched ones.
