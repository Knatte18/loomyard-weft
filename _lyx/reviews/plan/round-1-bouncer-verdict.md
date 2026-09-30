---
verdict: APPROVED
rationale: "All six round-1 findings (F1 retry script, F2 Publish marker-write and output-file assertions, F3 io.Discard mapped to nil stdio, F4 no conditional TestMain, F5 stray Uses entry, F6 shared-infrastructure sentence) are fixed in the current cards; every Decision and Constraint in the decision record is carried, and the verify section runs untagged and integration tests over every targeted package."
---

# Plan-Review round 1 — judge verdict

The round-1 report was itself `APPROVED`, with one MEDIUM, three LOW and two NIT findings.
I checked each fix against the current card text:

- **F1:** card 6's retry scenario scripts both merge-ins as tree-changed, with the first verify passing, the second failing and `Merge` refusing first with `ErrMergeInRequired`.
  It asserts two runner calls, Stuck, one `Merge` call, and no `PushBranch` or `MarkTaskDone` call.
  This matches `Finalize.Call`'s two `mergeInStep(ctx)` call sites (`internal/landingshed/finalize.go:159`, `:186`).
- **F2:** card 5 asserts that the output file holds the line the fake runner wrote, and it adds a tree-changed marker-write-failure case (returned error, not Stuck, no push, runner not called).
- **F3:** card 1 leaves `cmd.Stdout`/`cmd.Stderr` nil when `out` is `io.Discard`, so card 2's claim of unchanged behaviour now holds.
- **F4:** card 1 states outright that the package needs no `TestMain`, and gives the reason.
- **F5:** card 3's `Uses` names only `LoomScratchDir`.
- **F6:** card 1 adds `internal/verifyrun` to the shared-infrastructure sentence at `docs/overview.md:412` as well as to the package tree.

## Independent checks

- **Helpers the cards name:** `recordingResolver`, `newTestDeps`, `writeSummary`, `newPublishGitHubServer`, `newApprovalFixture`, `recordingParentMerger`, `newFinalizeDeps` and landingshed's `testmain_integration_test.go` all exist.
  `stuckOrCancelled` and `cancelErr` exist with the shapes the cards assume, and `mergeInStep` returns the `done` flag that card 6 sets.
- **Webster regression test:** `TestRunVerifyCommand` asserts only the `INFO`, `exitCode` and `WARN` substrings (and `cause`).
  `verifyrun.Run`'s log lines keep those substrings, so the test passes unchanged, as card 2 claims.
- **Spec section:** `contracts/specs/loom-plan-spec.md` has the `## verify model` section that card 7 extends.
- **Verify coverage:** the `## verify:` section runs `go test ./...` plus `go test -tags integration` over every targeted Go package (`verifyrun`, `websterengine`, `loomengine`, `cmd/lyx`, `landingshed`, `loomcli`).
  It only compiles `smoke`, which the rubric exempts.
- **Granularity:** the cards split along independently testable units.
  They are the runner, the webster reroute, the scratch accessors, the gate, the two producer call sites and the loomcli wiring.
  The gate's type, constructor and method stay together in card 4.
- **Fidelity:** every Decision (trigger, both producers, closure-read command, empty-command warning, Stuck-versus-error split, the kept merge commit, the marker, output capture, told paths, context-bound execution, the shared runner, Finalize retry placement, unaffected paths) has a card that carries it.
  So does every Constraint.
  The only scope beyond the decision record is the `docs/overview.md` entries for the new package, which the project's same-commit docs rule licenses.
- **`ImpactSummary`:** every `Edit` card states a real blast-radius conclusion, not a restatement of its Intent.
  No card is typed `Custom`.

## Focus departures

The round-1 report has no `## Focus departures` section.
It answered each of the six focus directives in its `## Focus directive verdicts` section, and I ratify those answers.
