---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: handler
    location: internal/loomcli/prepush.go:280-288
    summary: The check never surfaces a move of the run's parent branch, so the entry's base and parent-branch compare is missing
    fix: In `internal/loomcli/prepush.go`, make `guard` also read the parent branch's verdict from the run-scope report. The run-scope reconcile judges that branch (`originTargets` keeps the branch named by the origin record's `ParentBranch`), but `ownBranches` drops it because its `Slug` is the parent's, and `reportOfRun` in `landingdeps.go` narrows the entries to the run. Keep the parent branch's outcome (the one whose `Branch` equals the run's origin-record parent branch, or the prime's when the parent is the prime), post one notice line naming it when origin moved it past what this machine holds (state `fast_forward` or `diverged`), and name it in the stop reason when it stops the run. Add a `TestPrePushCheckGuard` row for each.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/loomcli/wiring.go:213-230
    summary: No test pins how a stopped check reaches callers, so Spawn returning nil, Push returning the stop error and the commit-records push_stopped envelope are unproven
    fix: Add rows to the existing commit-status and commit-records tests. A `commitRecordsVerb` row over stub `commitStatusDeps` whose `Push` returns an error wrapping `errPushStopped` asserts the ok envelope with `committed` true and a `push_stopped` key. A `loomCommitStatusDeps` test, with the check's seams stubbed, asserts that `Spawn` returns nil and starts no `spawnStatusPush` when the check stops, and that `Push` returns an error satisfying `errors.Is(err, errPushStopped)`. Use stubs, not a hub fixture.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/loomcli/prepush.go:148-164
    summary: The skip closure has no test, though it alone keeps the check off pushes-off runs, the prime, paused runs and Finalize
    fix: Test the skip condition without the check's other seams. Extract the closure's body into a function taking its reads as parameters, or drive `newPrePushCheck`'s `skip` from a hub fixture, and add a table-driven test with a row each for pushes off, the prime, a pause-covered run, a run whose status stands at `loomshed.NameFinalize` and a run at another producer, which must not skip.
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/loomcli/prepush.go:253-257
    summary: A stopped entry names an own needs_replay branch as stopped though that state does not stop an entry
    fix: In `stopReason`, name only the own branches whose state is one of rewound, rewritten, diverged, gone, dirty, busy, live or anchor_stopped, which is `BranchState.stops()`'s set, instead of excluding level, ahead and fast_forward. Add a row with an own `needs_replay` branch beside a stopping shared branch, which must name the shared-branch fallback.
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/loomcli/prepush.go:243-250
    summary: A failed read of whether the parent holds the run's work pauses the run, while every other failed read lets the push go
    fix: In `stopReason`, when `c.landed()` returns an error, keep the Warn and return `"", false` so the push goes and its own failure is recorded, as for an unreachable origin or a failed reconcile. Update the `stopReason` and `guard` doc comments, and add a `landed` error row to `TestPrePushCheckGuard` expecting no stop.
  - id: F6
    severity: LOW
    class: scope
    origin: handler
    location: internal/loomcli/wiring.go:209-212
    summary: The check runs after the status commit, but the entry asks for it before a run commits for a push
    fix: Run `check.guard()` also in `loomCommitStatusDeps`'s `Commit` seam, before `fabricengine.CommitAnchoredPaths`, and skip the commit when it stops, so a run another machine owns commits nothing onto records origin has moved. If keeping the commit local is the intended design, record that decision with `lym loom decision add` instead and leave the code. The three pushes then keep their own guard.
  - id: F7
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/loomcli/prepush_test.go:343-408
    summary: Table rows are missing for a failed notice post, several moved or stopped own branches, and a report entry of another slug
    fix: Add rows to `TestPrePushCheckGuard`. One has a `notify` that returns an error and expects the stop to hold. One has two own branches moved and expects both names joined by a comma in one notice. One has a stopped entry with an own `diverged` branch beside own `level` and `ahead` branches and expects only the diverged branch named. One has an entry of another slug in `report.Entries` and expects no stop.
  - id: F8
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/loomcli/landingdeps.go:339-342
    summary: PR-Review's PushTask wiring of the check and PausePair have no test
    fix: Add a `TestPausePair` beside `TestPauseOwnedElsewhere_PausesTheSlugOnceUnderTheOwnerCaller` in `internal/tendercli/cli_test.go` that asserts a plain pair entry with the default caller and no second entry when one stands. Add a `pullRequestDeps` test, or a row in an existing landing-deps test, that asserts `PushTask` returns the check's stop error and does not call `PushPairAnchored` when the check stops.
  - id: F9
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: pattern/PATTERN-owner-machine.md:3
    summary: The background file's verdict sentence and its Origin reconcile first section still name a resume, a wake and a landing but no push
    fix: In `pattern/PATTERN-owner-machine.md`, add a push to the sentence that says the owner verdict gates a resume, a wake or a landing and is judged inside `ReconcileOrigin`, and add one sentence to `## Origin reconcile first` saying a loom run's push of its pair starts with the run-scope origin reconcile too.
  - id: F10
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: pattern/PATTERN-owner-machine.md:38
    summary: The Enforcement line says loom tests reach loom's pre-push stops, but the only test drives the check over stubs
    fix: After the tests F2, F3 and F8 add, keep the line as written. If any of them is not added, change `loom's pre-push stops` in `pattern/PATTERN-owner-machine.md` to name the verdict mapping that `TestPrePushCheckGuard` pins.
---

### [MEDIUM] The check never surfaces a move of the run's parent branch

**Location:** internal/loomcli/prepush.go:280-288
**Issue:** The board entry says lym compares the run's base and parent branch before a push and, when only origin's tips moved, takes the new base or stops naming what moved. The run-scope reconcile judges the parent branch, but `ownBranches` keeps only branches whose `Slug` is the run's, and `reportOfRun` narrows the entries to the run. The parent branch's state therefore reaches neither a notice nor a stop reason. The owner compare of the beam and root is covered through `OwnerOf`'s effective owner.

### [MEDIUM] No test pins how a stopped check reaches callers

**Location:** internal/loomcli/wiring.go:213-230
**Issue:** `TestPrePushCheckGuard` drives `guard()` only. `Spawn` returning nil with no push started, `Push` returning the stop error and the `push_stopped` envelope in `commitRecordsVerb` are behaviours the diff adds and no test reaches. They are how a stop becomes visible to the run's callers.

### [MEDIUM] The skip closure has no test

**Location:** internal/loomcli/prepush.go:148-164
**Issue:** The test replaces `skip` with a stub. The closure alone keeps the check off pushes-off runs, the prime, a paused run and Finalize, so a regression there pauses a run mid-landing with no test failing.

### [LOW] A stopped entry names an own needs_replay branch as stopped

**Location:** internal/loomcli/prepush.go:253-257
**Issue:** The filter keeps every state but level, ahead and fast_forward, so needs_replay, which `BranchState.stops()` does not count, is named as a branch that stopped the run when a shared branch stopped it.

### [LOW] A failed read of the parent's holding of the run's work pauses the run

**Location:** internal/loomcli/prepush.go:243-250
**Issue:** A failed `landed()` reads as not landed, so an ended entry pauses the run. A failed check, an unreachable origin and a failed reconcile all let the push go, so this one differs and can pause a healthy run on a transient error.

### [LOW] The check runs after the status commit, not before it

**Location:** internal/loomcli/wiring.go:209-212
**Issue:** The entry names "before a run commits for a push". The guard sits in the push seams, so a stopped run has already committed to records. The commit stays local, which may be intended; the diff records no such decision.

### [LOW] Table rows are missing for several guard paths

**Location:** internal/loomcli/prepush_test.go:343-408
**Issue:** No row has a failed notice post, several moved or stopped own branches, or a report entry of another slug, so the filters at `prepush.go:232`, `:254` and the join in `movedBranches` are unpinned.

### [LOW] PR-Review's PushTask wiring and PausePair have no test

**Location:** internal/loomcli/landingdeps.go:339-342
**Issue:** No test reaches `pullRequestDeps`' `PushTask` guard, and `cli_test.go` covers `PauseOwnedElsewhere` only, not the new `PausePair`.

### [LOW] The pattern background file leaves its verdict sentence and reconcile section without a push

**Location:** pattern/PATTERN-owner-machine.md:3
**Issue:** The new bullets name the push, but the sentence listing what the origin verdict gates and the `## Origin reconcile first` section still name a resume, a wake and a landing.

### [LOW] The Enforcement line overstates what the tests reach

**Location:** pattern/PATTERN-owner-machine.md:38
**Issue:** It says the loom tests reach loom's pre-push stops, but only the verdict mapping over stubs is tested.

## Rejected

- Origin unreachable making every status transition wait on a network call (handler): the reconcile runs with zero-value `ReconcileOptions`, which means one attempt bounded by the attempt deadline; no evidence of a long stall was found.
- A dirty own code branch pausing every push (handler): `ffWorktree` marks a branch dirty only when a dirty tracked path overlaps what origin touched, as the correctness lens confirmed.
- A tests-duplication finding (handler): no new test covers behaviour an existing test covers, and `TestPrePushCheckGuard` is the only test added.
- An unguarded loom-run push path (lens:code-affected-set): none exists; `pawlwiring.go:94` and the batten paths are not a loom run's push.
