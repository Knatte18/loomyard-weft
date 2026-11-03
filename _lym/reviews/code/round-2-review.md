---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: internal/loomcli/prepush.go:28
    summary: codeRepo repeats the literal "code" that fabricengine declares as its unexported failedPushSideCode, against PATTERN-named-constants
    fix: In internal/fabricengine/failedpush.go export the repository-side value BranchOutcome.Repo carries for the code repository as a documented constant (for example RepoCode, set equal to failedPushSideCode, or rename failedPushSideCode and update its uses). Delete codeRepo from internal/loomcli/prepush.go and use the exported constant in parentOutcome and in internal/loomcli/prepush_test.go, which builds BranchOutcome values with the literal "code".
  - id: F2
    severity: BLOCKING
    class: consistency
    location: internal/loomcli/prepush.go:75
    summary: the parent-notice record path is built in loomcli from a local file name, while PATTERN-shed-driver keeps run paths only in internal/shedrun
    fix: Add a path function beside LandedCommitFile in internal/shedrun (for example ParentNoticeFile(l, runID) returning filepath.Join(ScratchDir(l, runID), "pre-push-parent-notice")), with a test in internal/shedrun/paths_test.go next to the other scratch paths. Delete parentNoticeFile from internal/loomcli/prepush.go and have newPrePushCheck's parentNoticed seam call the shedrun function.
  - id: F3
    severity: MEDIUM
    class: scope
    location: internal/loomcli/wiring.go:200
    summary: no test pins that loomCommitStatusDeps and pullRequestDeps.PushTask are wired through the check, or that the skip closure in newPrePushCheck wires its reads correctly
    fix: Add one test in internal/loomcli that builds loomCommitStatusDeps over a hub fixture (hubforge) whose origin record names an owner elsewhere, and asserts Commit returns an error satisfying errors.Is(err, errPushStopped), Spawn returns nil and Push returns an error satisfying errors.Is(err, errPushStopped) without a push. Add a case for pullRequestDeps(...).PushTask returning the stop. Drive newPrePushCheck(...).skip itself for a pushes-off environment and for the prime, instead of only skipsCheck with stubs. Fold these into existing wiring tests where one covers the same constructor.
  - id: F4
    severity: LOW
    class: consistency
    location: internal/loomcli/prepush.go:235
    summary: stoppedBranches names an own records branch that is busy as a stopping branch, which classifyEntries exempts for the run scope's own pair
    fix: In stoppedBranches skip a branch with Repo records and State BranchBusy, the exemption classifyEntries applies at internal/fabricengine/originreconcile.go:864, so a stop caused by a shared branch reads "a branch every run shares" instead of naming the run's busy records branch. Add a row to TestPrePushCheckGuard with an own busy records branch beside a stopped entry.
  - id: F5
    severity: LOW
    class: consistency
    location: pattern/PATTERN-owner-machine.md:7
    summary: the new bullet says a push proceeds only after the reconcile reads the entry as active, but the code also lets it go on a failed reconcile, a failed landed read, an ended entry whose parent holds the work, pushes off and the prime
    fix: In the bullet "A loom run's commit and push of its pair proceed only after a run-scope origin reconcile reads its entry as active" extend the exception list to match prepush.go's guard doc comment, naming pushes off, the prime, a failed or unreachable reconcile and an ended entry whose parent holds the run's work, or say the reconcile judges and the push goes on any failure of the check itself.
---

### [BLOCKING] codeRepo repeats a fabricengine literal

**Location:** `internal/loomcli/prepush.go:28`
**Issue:** `BranchOutcome.Repo` is `"code"`, `"records"` or `"bolt"`, set from fabricengine's unexported `failedPushSideCode`. loomcli redeclares `"code"` as `codeRepo`, and the test builds outcomes with the bare literal. A rename on the fabric side leaves `parentOutcome` finding no parent branch with no failure, so the parent notice and the parent name in a stop go silent. `PATTERN-named-constants` asks for one declaration referenced everywhere.

### [BLOCKING] Run path built outside internal/shedrun

**Location:** `internal/loomcli/prepush.go:75`
**Issue:** `PATTERN-shed-driver` keeps run paths only in `internal/shedrun`. `LandedCommitFile` shows the convention: a named file under `ScratchDir` gets its own function there. `newPrePushCheck` joins `ScratchDir` with `parentNoticeFile` itself.

### [MEDIUM] Wiring of the check to its seams is unpinned

**Location:** `internal/loomcli/wiring.go:200`
**Issue:** `TestPrePushCheckGuardedSeams` pins the `guarded*` helpers over stubs, and the commit-records tests pin the envelope. Nothing pins that `loomCommitStatusDeps` wraps `Commit`, `Spawn` and `Push` in them, that `pullRequestDeps.PushTask` is wrapped (the diff adds no test of the PR-Review push), or that the `skip` closure in `newPrePushCheck` passes the right reads to `skipsCheck`. Dropping a wrapper, or swapping `SkipPush` for another option, fails no test. `TestSkipsCheck` drives `skipsCheck` with stubs, not the closure.

### [LOW] Own busy records branch named as a stop

**Location:** `internal/loomcli/prepush.go:235`
**Issue:** For the run scope's own pair, `classifyEntries` ignores a busy records branch because the run's driver holds it. `stoppedBranches` keeps it, since `BranchBusy.Stops()` is true. When a shared branch stops the entry, the notice then names the run's own busy branch and omits the shared-branch wording the tests expect for `needs_replay`.

### [LOW] PATTERN bullet narrower than the code's skips

**Location:** `pattern/PATTERN-owner-machine.md:7`
**Issue:** The bullet lists Finalize, a pause and an unreachable origin as the exceptions. `guard` also goes ahead when pushes are off, in the prime, on a failed reconcile, a failed skip read, a failed `landed()` read, and an ended entry whose parent holds the work. The verdict sentence and `## Origin reconcile first` section match the code. Only the bullet's exception list under-states it.

## Rejected

- A parent branch that origin moved reaches only a notice, never a stop: the run's own entry is classified from its own branches, and the landing's Parent-Align takes the new parent base. `TestPrePushCheckGuard` pins both the notice rows and the parent branch named inside an already-stopped entry. Not a defect against the board entry's "takes the new base ... or stops".
- Failed `landed()` read, failed notice post and own `needs_replay` branch: each lets the push go or stops consistently with the other failed reads, and `needs_replay` is outside `BranchState.Stops()`, matching its exclusion from `stoppedBranches`.
- Ordering: `guardedCommit` runs the check before `CommitAnchoredPaths`, so "before a run commits for a push" is met.
