---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    location: _lyx/plan/06-finalize-verify-gate.md
    summary: the merge-in-required retry test scripts an up-to-date first merge-in with no marker, so the runner runs once, yet the card asserts it runs a second time
  - id: F2
    severity: LOW
    location: _lyx/plan/05-publish-verify-gate.md
    summary: two Publish acceptance criteria (marker write or output-file create failure is a returned error with no push; the output file holds the command output) are asserted only at gate level, never through Publish.Call
  - id: F3
    severity: LOW
    location: _lyx/plan/01-verifyrun-runner.md
    summary: passing io.Discard to os/exec makes a copy pipe where webster today hands the child the null device, so a lingering child now adds up to the 10s WaitDelay and routes through ErrWaitDelay, contradicting card 2's identical-behaviour claim
  - id: F4
    severity: LOW
    location: _lyx/plan/01-verifyrun-runner.md
    summary: card 1 conditionally creates a testmain_test.go that is not among its Create targets, on a condition that cannot fire and an invariant that does not bind
  - id: F5
    severity: NIT
    location: _lyx/plan/03-loomengine-verify-paths.md
    summary: card 3 lists loomApprovalFileName under Uses although the new accessors never consume it; they only declare constants beside it
  - id: F6
    severity: NIT
    location: _lyx/plan/01-verifyrun-runner.md
    summary: card 1 adds verifyrun to docs/overview.md's package tree but not to the shared-infrastructure sentence that already names its sibling leaf internal/shell
---

# Plan-Review round 1

The plan carries every Decision and Constraint in the decision record: the gate trigger, both producers, the closure-read command, the empty-command warning, Stuck-versus-error classification, the marker, told paths, in-process context-bound execution, the shared runner, the placement inside `mergeInStep`, and the unaffected paths.
Docs land where the record says (`internal/landingshed/doc.go` in cards 5 and 6, the spec's `## verify model` in card 7).
The `## verify:` section runs `go test ./...` plus `-tags integration` over every targeted package.
The findings below are test-specification and precision defects, none of which blocks the plan.

### [MEDIUM] Card 6's retry test contradicts its own script

**Location:** `_lyx/plan/06-finalize-verify-gate.md`, the last assertion paragraph.
**Issue:** The card scripts the resolver to return up-to-date on the first call and tree-changed on the second, with no marker present, then asserts "the runner runs a second time after that merge-in".
With an up-to-date first merge-in and no marker, the gate returns at step 2 of card 4's `check` without calling the runner, so the runner runs exactly once across the whole call.
An implementer following the card literally writes a call-count assertion of 2 that fails, or silently rewrites the script.
The decision record's criterion says the gate "runs again" after the retry's merge-in, which presumes a first run.
**Fix:** Script both merge-ins as tree-changed, the first verify passing and the second failing, with the merger returning `*fabricengine.ErrMergeInRequired` first.
Assert the runner is called twice, the verdict is Stuck, and `Merge` is called exactly once.

### [LOW] Two Publish acceptance criteria are asserted only at gate level

**Location:** `_lyx/plan/05-publish-verify-gate.md`, the `publish_verify_test.go` list.
**Issue:** The decision record's criteria "`Publish`, marker write or output-file create failure: a returned error, not Stuck; no push" and "the output file holds the command's output" (on the failing Publish case) have no assertion through `Publish.Call`.
Card 4 covers the marker and output-file failures in `check` and the file contents with the real runner, but nothing proves `Publish` maps a gate infrastructure error to a returned error with no push, and the closure-error case exercises the same mapping only indirectly.
**Fix:** Add a marker-write-failure case to card 5 (the marker path under a regular file; a returned error, not Stuck, no push, runner not called).
Have the failing fake runner write a known line into its writer and assert the output file holds it.

### [LOW] `io.Discard` changes webster's teardown against today

**Location:** `_lyx/plan/01-verifyrun-runner.md` (the `cmd.Stdout`/`cmd.Stderr` sentence) and `_lyx/plan/02-webster-shared-runner.md` (the behaviour claim).
**Issue:** Today `runVerifyCommand` leaves `cmd.Stdout`/`cmd.Stderr` nil, so os/exec hands the child the null device.
`os/exec`'s `writerDescriptor` special-cases only nil and `*os.File`, so `io.Discard` gets a pipe plus a copy goroutine.
A verify line that leaves a child holding stdout now makes `Wait` block up to the 10-second `WaitDelay` and return `exec.ErrWaitDelay`, a path webster never took.
Pass/fail stays correct through the `ProcessState` classification, but card 2's claim that the bisect sees identical behaviour does not hold.
**Fix:** In card 1, map an `out` of `io.Discard` to nil `cmd.Stdout`/`cmd.Stderr` so the child writes to the null device with no copy pipe.
Webster keeps passing `io.Discard`, as the decision record says, and its spawn behaviour stays as today.

### [LOW] Card 1's conditional `testmain_test.go`

**Location:** `_lyx/plan/01-verifyrun-runner.md`, "If `cmd/lyx`'s hermetic-environment scan flags the new test package…".
**Issue:** The card instructs creating a file on a runtime condition, and that file is not among its Create targets.
The condition cannot fire: `TestHermeticGitEnv_GitSpawningPackagesHaveTestMain` flags a package only on the tokens `gitexec.Run`, `exec.Command`, `gitkit.Copy`, `gitkit.MustRun`, `gitkit.SeedConfig` and `hubforge.NewHub`, and a test that calls `verifyrun.Run` carries none of them.
The Hermetic Git Test Environment Invariant binds only packages whose tests spawn git, and this one spawns `sh`/`cmd`.
**Fix:** Replace the conditional with a determinate statement: the package needs no `TestMain`, because its tests spawn no git and carry no git-spawn token.

### [NIT] Card 3's `Uses` names a symbol it does not consume

**Location:** `_lyx/plan/03-loomengine-verify-paths.md`, `**Uses:**`.
**Issue:** The new accessors are built on `LoomScratchDir` and their own new constants.
They never reference `loomApprovalFileName`, which the Intent names only as the placement anchor.
**Fix:** Drop `internal/loomengine#loomApprovalFileName` from `Uses`.

### [NIT] Card 1's overview edit misses the shared-infrastructure sentence

**Location:** `_lyx/plan/01-verifyrun-runner.md`, the last Intent paragraph.
**Issue:** `docs/overview.md` also lists the shared infrastructure leaves in one sentence ("The user-facing modules sit on a thin layer of shared infrastructure (… `internal/shell` …)").
`verifyrun` is the same kind of mechanical, domain-free leaf shared by two modules, so adding it to the package tree alone leaves the two listings inconsistent.
**Fix:** Add `internal/verifyrun` to that sentence's list as well.

## Focus directive verdicts

- **Cards 1 and 2 contract mapping:** not a defect.
  With a background context the cancellation branch never fires.
  A non-zero exit yields `(n, nil)` → `(false, nil)` with an `Info` line carrying `exitCode` and no `WARN`.
  A cleared `PATH` makes `exec.CommandContext`'s `LookPath` fail, so the error reaches the spawn-failure branch, which logs a `Warn` with `cause` and returns `(-1, err)` → a wrapped error.
  `TestRunVerifyCommand` asserts only the `INFO`/`exitCode`/`WARN`/`cause` substrings, never the renamed message prefix or the `verifyCmd`/`worktree` keys, so it passes unchanged (F3 concerns timing, not these assertions).
- **Card 5 Publish criteria:** the two gaps are F2; every other Publish criterion has a named assertion.
- **Card 6 retry and Finalize criteria:** the scenario's shape is F1.
  Both `Call` sites reach `mergeInStep` as the card says.
  "Parent branch unchanged", "`MarkTaskDone` not called" and "nothing pushed" are asserted through `Merge`/`PushBranch` never being called and a recording `MarkTaskDone`.
- **Card 7 setup and drift guard:** not a defect.
  With only `HubPath` set, `AnchorPath()` is `filepath.Join(HubPath, "", "")` = `HubPath`, and the closure and the test both use `planparser.PlanDir(l.AnchorPath())`.
  `TestLandingDeps_EveryFieldPopulated` reflects over every field with `IsZero`, so a nil func or an empty string among the three new fields fails it without an edit.
- **Docs for the new leaf:** the overview package-tree line is licensed by the project's same-commit docs rule (a new package changes the tree), not new scope.
  The shared-infrastructure sentence is F6.
  `docs/shared-libs/README.md` is not exhaustive (it omits `internal/shell` and `internal/gitkit`), so leaving `verifyrun` out of it is not a defect.
  The Told-Geometry bound list pairs each entry with an enforcement test, so adding `verifyrun` to it would add scope the decision record does not license; not a defect.
- **Conditional `testmain_test.go`:** F4.
