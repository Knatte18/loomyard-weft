---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    location: internal/landingshed/deps.go:121
    summary: The three new Verify* field comments are fixed-column wrapped mid-sentence, against the Comment Line-Break Convention
  - id: F2
    severity: LOW
    location: internal/landingshed/verifygate.go:1
    summary: The verifygate.go file header and the second paragraph of check's doc comment are fixed-column wrapped mid-sentence
  - id: F3
    severity: LOW
    location: internal/landingshed/publish.go:146
    summary: The new Step 4a comment in Publish.Call is fixed-column wrapped mid-sentence
  - id: F4
    severity: LOW
    location: internal/landingshed/doc.go:10
    summary: doc.go's changed Publish sentence, the added Publish paragraph and the Finalize edit leave mid-sentence breaks, including an orphaned split of The landing commit carries the change
  - id: F5
    severity: LOW
    location: internal/loomengine/config.go:208
    summary: The LoomVerifyPendingPath and LoomVerifyOutputPath doc comments break mid-sentence after is ephemeral
  - id: F6
    severity: LOW
    location: internal/loomcli/landingdeps.go:97
    summary: The VerifyCommand closure comment breaks mid-phrase after never at
  - id: F7
    severity: LOW
    location: internal/landingshed/publish_verify_test.go:1
    summary: New test-file comments in four files (verifygate_integration_test, publish_verify_test, finalize_verify_test, landingdeps_verify_test) are fixed-column wrapped mid-sentence
  - id: F8
    severity: NIT
    location: internal/verifyrun/run_integration_test.go:55
    summary: The Discard subtest uses a custom discard type described as io.Discard by value, which it is not; it duplicates the Buffer path while a separate test and helper cover the real io.Discard
  - id: F9
    severity: NIT
    location: internal/landingshed/publish_verify_test.go:28
    summary: publishVerifyFixture.ghCalls is declared and never used
  - id: F10
    severity: NIT
    location: internal/landingshed/verifygate_test.go:101
    summary: TestVerifyGate_TreeChangedPass asserts only that the runner got a non-nil writer, not that it got the told output file Card 4 asks the tests to check
---

## Review range

`_lyx/shed/publish-verify-after-merge/status.json` gives `product.parent: main`.
The range is `git diff $(git merge-base main HEAD)..HEAD` with merge base `6b41264db`: the seven card commits `18c301da5`..`009d49c02`, plus two parent merge-ins that add nothing over the merge base.

## Substrate

I ran every command in the plan's `## verify:` section, scoped to the touched packages: `go build ./...`, `go vet -tags integration` and `go vet -tags smoke` over the listed packages, `go test` untagged and with `-tags integration` over `internal/verifyrun`, `internal/websterengine`, `internal/landingshed`, `internal/loomengine`, `internal/loomcli` and `cmd/lyx`.
All pass.
I also ran `internal/websterengine`'s `TestRunVerifyCommand` verbosely under `-tags integration`, and it passes unchanged.

## Verdicts on the focus directive's questions

**verifyGate.check against Card 4's ten steps.**
Every branch matches.
The nil command returns before the stat.
The stat error other than not-exist is returned.
The no-change, no-marker path returns before the closure runs.
A closure error is returned.
The empty command warns with the producer and parent branch, adds the marker path only when present, and neither writes nor removes the marker.
The marker is written only when `treeChanged`, after `MkdirAll`.
The output file is created after `MkdirAll`.
A cancelled context yields a returned error wrapping `ctx.Err()`.
A spawn failure and a non-zero exit yield Stuck reasons, and neither touches the marker.
Only exit 0 removes the marker, tolerating not-exist.
The output-file `Close` error is logged only when `runErr` is nil, and the run then continues as a pass or a failure by exit code.
That is acceptable: the file only carries diagnostics, the card specifies no Close handling, and a lost tail on a failed run still leaves the Stuck reason and exit code intact.
Not a defect.

**verifyrun.Run against Card 1.**
The classification order is right: `err == nil` returns first, then `ctx.Err()` is checked before `ErrWaitDelay` and `*exec.ExitError`, and the spawn-failure fallthrough wraps the cause and names the command.
The `io.Discard` comparison leaves `Stdout` and `Stderr` nil, as the card requires.
Log lines: Info on spawn, Info with `exitCode` on every exit path (0, `ErrWaitDelay`, `ExitError`), Warn with `cause` on a spawn failure, and Info on cancel.
That meets the Live-Substrate Spawn Observability rule.
`runverify_test.go` asserts only `INFO`, `exitCode`, `WARN` and `cause`, never the old prefix or the `verifyCmd`/`worktree` keys, so the key rename does not break it; the verbose run above confirms it.

**Gate placement.**
In `Publish.Call` the gate sits right after the resolver's `OutcomeStuck` return and before step 5's push, the owner/repo parse and `NewGitHubClient`.
In `Finalize.mergeInStep` it sits after the `OutcomeStuck` return, and both of `Call`'s merge-in sites go through `mergeInStep`, so it precedes `parentOpener`, the parent-side `Merge`, `MarkTaskDone`, `PushBranch` and the pull-request close.
Errors go through `cancelErr` and then get wrapped with the producer name; a reason goes through `stuckOrCancelled`, with `done=true` in Finalize.
Matches Cards 5 and 6.

**loomcli wiring.**
`planparser.PlanDir(l.AnchorPath())` is the same resolution `loomengine/plan.go`, `loomcli/wiring.go`'s approval and `hubgeom.WebsterGeometry` use, so landing reads the same plan the rest of loom reads.
`WorktreeRoot` is `l.WorktreePath()`, the same directory `hubgeom.WebsterGeometry` gives webster's bisect, so the command runs where webster already runs it.
Every loom path that reaches `Publish` or `Finalize` passes `Plan-Write` first in `contracts/recipes/loom-recipe.yaml`, and no row between them archives or removes the plan.
The approval and no-PR paths return before the merge-in and never call the closure.
So a missing plan cannot turn an ordinary landing into a returned error.
Not a defect.

**Test coverage and tiers.**
Each card's enumerated cases are present.
`run_integration_test.go` covers exit 3 with both a buffer and `io.Discard`, a cleared `PATH` with WARN and `cause`, and cancellation bounded well under `sleep 30`.
`verifygate_test.go` covers every numbered branch.
`publish_verify_test.go` has the failing `NewGitHubClient` factory, the marker-write failure, and the no-PR and approval paths.
`finalize_verify_test.go` covers the retry: two runner calls, one `Merge`, no push, no `MarkTaskDone`.
`landingdeps_verify_test.go` writes the plan after construction, checks the ` && ` join and checks `("", nil)` for a missing section.
The untagged files spawn nothing: fakes, `t.TempDir()` files and the package's existing `httptest` GitHub server only.
The only gap is F10, below.

**Per-card mechanical checks.**
For the Create groups, the only `"sh", "-c"` / `"cmd", "/C"` literal left in non-test code is `internal/verifyrun/run.go`, so no second runner survives in websterengine or elsewhere.
Nothing outside the new accessors, `landingshed` and `loomcli/landingdeps.go` names the verify marker.
For the Edit groups, Card 3 touches only `TestLandingAndApprovalAccessors`, `transientSet` and the two `TestConstructorAnchoring_*` functions, each with the two assertion lines, and Card 4 adds one `landingshedAllowedImports` entry.
Every Edit group stays inside its card's ImpactSummary.

## Findings

### [LOW] Verify* field comments in Deps keep a fixed-column wrap

**Location:** internal/landingshed/deps.go:121-136
**Issue:** CONSTRAINTS.md's Comment Line-Break Convention requires one sentence per line in new comments.
The three new field comments break mid-sentence ("the task\n// worktree", "because the\n// caller", "the gate never\n// runs", "no verify\n// command", "the verify has not yet passed. Told, never derived, per the\n// Told-Geometry Invariant", "written to,\n// overwritten").
**Fix:** Rewrite the three comments one sentence per line.

### [LOW] verifygate.go header and check doc keep a fixed-column wrap

**Location:** internal/landingshed/verifygate.go:1-3, 293-294
**Issue:** The file header breaks mid-sentence ("their parent\n// merge-in", "as a\n// Stuck reason"), and so does the first sentence of check's second paragraph ("an infrastructure\n// fault").
**Fix:** Rewrite both with semantic line breaks.

### [LOW] Step 4a comment keeps a fixed-column wrap

**Location:** internal/landingshed/publish.go:146-148
**Issue:** "A merge can compile\n// cleanly" and "the plan-level verify\n// already passed" break mid-sentence.
**Fix:** One sentence per line.

### [LOW] doc.go edits leave mid-sentence breaks

**Location:** internal/landingshed/doc.go:10-12, 19-20, 35-36
**Issue:** The changed Publish line ("verifies the merged tree, …") continues a sentence that still wraps at a fixed column onto the next lines.
The added Publish paragraph breaks "left by an\n// earlier unverified merge".
The Finalize edit splits the untouched sentence "The landing commit carries the change description and exactly one Co-Authored-By trailer…" into a stub line and a continuation, a break the edit created.
**Fix:** Put the changed Publish sentence on semantic breaks from "Without an approval" through "landing commit)".
Put the added paragraph's first sentence on one line (or break it at the clause boundary).
Rejoin "The landing commit carries the change description…" as one line.

### [LOW] LoomVerify*Path doc comments break mid-sentence

**Location:** internal/loomengine/config.go:426-427, 435-436
**Issue:** "and is ephemeral: the\n// marker is never tracked" (and "the\n// file") is a mid-phrase break in a new comment.
**Fix:** Put each sentence on one line.

### [LOW] VerifyCommand closure comment breaks mid-phrase

**Location:** internal/loomcli/landingdeps.go:97-99
**Issue:** "never at\n// construction" breaks mid-phrase in a new comment.
**Fix:** Break at the colon's clause boundaries instead.

### [LOW] New test-file comments keep a fixed-column wrap

**Location:** internal/landingshed/verifygate_integration_test.go:3-4, internal/landingshed/publish_verify_test.go:1-2, 21-22, internal/landingshed/finalize_verify_test.go:1-2, 124-125, internal/loomcli/landingdeps_verify_test.go:1-3
**Issue:** The convention covers every Go comment, test files included, and these new comments break mid-sentence ("echo-and-exit\n// commands", "between the\n// parent merge-in", "a fake resolver and a\n// push closure", "after\n// every parent merge-in", "second\n// merge-in", "loomengine's\n// accessors").
**Fix:** Rewrite them one sentence per line.

### [NIT] Discard subtest misdescribes and duplicates coverage

**Location:** internal/verifyrun/run_integration_test.go:55-72, 132
**Issue:** The `Discard` subtest passes a custom `discard{}` type whose comment calls it "io.Discard by value", which it is not.
It exercises the same general-writer path as the `Buffer` subtest.
The real `io.Discard` case lives in a separate `TestRun_IoDiscard` behind a pointless `ioDiscard()` helper.
**Fix:** Pass `io.Discard` directly in the `Discard` subtest, and drop the custom type, `TestRun_IoDiscard` and the helper.

### [NIT] Unused fixture field

**Location:** internal/landingshed/publish_verify_test.go:28
**Issue:** `ghCalls` is never read or written.
**Fix:** Remove it.

### [NIT] Pass test does not pin the writer the runner received

**Location:** internal/landingshed/verifygate_test.go:101-103
**Issue:** Card 4 asks for exact assertions on the writer the fake receives.
The test only checks that it is non-nil, so a gate that handed the runner some other writer would pass.
**Fix:** Assert the writer is an `*os.File` whose `Name()` is the told output path.
