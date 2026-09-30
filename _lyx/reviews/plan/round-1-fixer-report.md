# Plan-Review round 1 — fixer report

I fixed all fourteen findings.
The plan now has 14 cards instead of 11: card 2 is split into two cards, card 9 is split into two, and a new producer card is added.
The cards are renumbered 1..14, and every in-card reference, commit prefix and Card Index row matches the new numbers.
`lyx loom validate-plan` reports `ok: true` with only informational findings.
While fixing F4, the validator flagged a `plan:internal/shuttleengine#ErrPaneNotReady` handle that only card 13 used (`handle-unreferenced`).
I dropped the handle and declared the sentinel in card 13's Intent under its `run.go` edit.

## Changes

- **F1** (no transient class reachable): new card 8, `08-producer-transport-errors.md`.
  Publish's push, pull-request query and pull-request create arms, and Finalize's parent-push arm, now return a `%w`-wrapped hard error when `shedtransient.Class` classifies the failure.
  Every other failure keeps its `Stuck` verdict.
  The card also covers `landingshed`'s allowlist, fake-based tests for both branches, and the two existing 500-fixture tests moving to 422.
  The overview gains a shared decision, "A transient failure is an error, never a verdict", and `./internal/landingshed/...` joins both tagged verify lines.
  Card 7's Transient Stop Invariant text adds the error-not-verdict rule.
- **F2** (bootstrap push claim): card 7 (was 6) drops the false seed-and-commit push claim and the `integration_transientbootstrap_test.go` Create.
  It now states that no bootstrap sub-step reaches a remote, so the `PreStep` mark is inert today.
  Real-remote coverage moved to card 2's `transport_integration_test.go`, which pins real git stderr against `http://127.0.0.1:1/...` and a `pre-receive` reject.
- **F3** (`-c` read as the subcommand): card 2 defines the subcommand as the first argument after git's global options, where `-c` and `-C` each consume the next argument.
  Test rows cover `-c push.autoSetupRemote=true push` and `-C <dir> fetch`.
  Card 5's table uses the `-c` form too.
- **F4** (duplicate resume sends): card 13 adds `shuttleengine.ErrPaneNotReady`, wrapped by `requireReadyAgentPane`'s not-ready refusal.
  `resumeParkedDriver` retries only that sentinel and refuses at once on any other error.
  New tests cover the sentinel wrap (`run_test.go`) and a one-call refusal for any other error.
  The overview gains a shared decision for this.
- **F5** (GitHub deadline): card 3 makes `IsTransient` true for `context.DeadlineExceeded` and keeps it false for `context.Canceled`, and its test rows are adjusted to match.
- **F6** (`pull` op): card 2's subcommand set is D2's four operations, and `pull` is now a false test row.
- **F7** (`lyx loom step` help text): card 6 edits `internal/loomcli/cli.go`'s `Step` `Long` help to name `transient`.
- **F8** (`last_step` doc): card 9 gains a `Prosa` target, `docs/overview.md`, extending the `last_step` sentence.
- **F9** (wrong card in ImpactSummary): card 1's ImpactSummary now names card 7.
- **F10** (card 2 granularity): split into card 2 (`gitexec`) and card 3 (`githubclient`).
- **F11** (card 9 granularity): `driverResumeLine` moved to its own card 12, in a new `driverresumeline.go` with its own test file.
- **F12** (untested `battenPreStep` wiring): card 7 extends the three `battenPreStep` refusal tests and loom's `TestStepCmd_BusyRefusal_BeforeBootstrap` to assert an empty `transient`.
- **F13** (batten acceptance criterion): card 13 names the awaiting→approve→`Spawn` half and maps it to a resume test asserting that the branch returns true with no refusal, which the `--no-attach` envelope depends on.
  Card 11 names the `done` half (batten's `callDone` waits for the strand that the teardown ends).
- **F14** (clean-tree path filter): card 14 states D5's plain porcelain rule, and explains that weft records never appear because the warp's `_lyx` and `.lyx` links are excluded.

## Departure from a written fix

F1's fix in the review listed batten's Worktree-Create arm.
While writing the card I read `internal/battencli/wire.go`'s `createRefusal` doc: "a rolled-back create keeps the branch it made".
So an immediate re-step after a transport failure there would stop again on `ErrBranchExists` rather than clear.
Card 8 therefore leaves Worktree-Create as a verdict and says why in its out-of-scope list.
This narrows the written fix; it does not defer it.

## Deferred

None.
