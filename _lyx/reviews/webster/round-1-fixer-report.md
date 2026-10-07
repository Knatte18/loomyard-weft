# Round 1 fixer report

All six findings are fixed, each in its own commit, none pushed.
After the last commit `go build ./...`, `go vet ./...` and `go test ./...` pass, `go test -tags integration` passes over the touched packages, `go test -tags tmux ./internal/reedengine/...` passes and `go vet -tags llm ./...` is clean.

## What changed

- **F1** (`1613c1bd2`): re-broke every changed or new Go comment the review listed at sentence and clause boundaries, in `internal/battenshed/innerrun.go`, `innerrun_test.go`, `internal/reedengine/lifecycle.go`, `windowsize.go`, `waitmark_integration_test.go`, `internal/reedcli/resume.go` and `internal/tokenvocab/doc.go` and `tokenvocab_test.go`.
  No untouched line was rewrapped.
- **F2** (`b5017f4e2`): added `t.Parallel()` to the new tests that run over fakes, temp directories or pure functions, in `loomcli`, `battencli`, `battenshed`, `reedengine` and `shuttleengine`, and verified them with `go test -race -count=3` over those five packages.
  Added a sentence atop `innerrun_halted_test.go` naming the process-global logger output the two log-capturing tests replace, so they stay serial.
- **F3** (`ec0c1adfd`): `verifyGateStuckReason` now names the verify timeout and the log path when the last report is a timed-out one, instead of `none named`.
  A new test, `TestVerifyGateStuckReason_TimedOutReportNamesTheTimeoutAndLog`, covers it, since no test reached the case.
  The `verify gate exhausted` row of `contracts/specs/refusal-spec.md` says the same.
- **F4** (`d2a38511e`): `callHalted` removes the revive once-marker whenever it starts a new halt episode by its halt-warned marker, so a re-halt at a new history length revives a dead driver again.
  A new row in `TestInnerRun_HaltedRevivesADeadDriverOncePerEpisode` fails without the change (0 revives) and passes with it.
- **F5** (`03a531702`): deleted `TestStrandNamed`, whose rows `TestAddStrandUnless_NamedStrandSkips` already asserts through `AddStrandUnless`.
  Added the one case the public test lacked, an orch strand behind another strand, as a row there.
- **F6** (`97457286d`): split the `docs/overview.md` loom bullet after its first sentence.

## Correction to the review

The F2 finding as first written also named `TestReadWaitMarker` as swapping `isAlive` with nothing naming the state.
That was wrong: `internal/shuttleengine/waitmark_test.go` already names the process-global seam in its file header.
I corrected F2's summary and body in the review file before fixing, and changed nothing in that file's tests.

## Deferred

Nothing.
