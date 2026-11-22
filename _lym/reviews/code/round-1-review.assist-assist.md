---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/battencli/commitstatus.go:263
    summary: A fresh start has no Worktree-Create marker, so Seed-Child still waits for a synchronous records push.
    fix: In internal/battencli/arm.go, establish the fresh Worktree-Create transition through the commit seam after seeding releases the status lock and before the first producer runs. Propagate commit and push failures. Preserve the missing-marker fallback for resumed runs. Extend the existing fresh-start test through the real seam with fake commit and push boundaries to assert that Seed-Child commits without pushing, rather than supplying a synthetic initial seam call.
    origin: lens:code-correctness
  - id: F2
    severity: MEDIUM
    class: design
    location: internal/battencli/commitstatus_test.go:80
    summary: No assertion checks that start transitions bypass network admission.
    fix: Extend TestNewCommitStatusSeam with a recording Admit callback and independent expected admission counts in its existing cases. Assert that start transitions acquire no admission and ordinary transitions still acquire and release it. Use the fake callback without a real gate or timed wait.
    origin: lens:code-tests
  - id: F3
    severity: LOW
    class: consistency
    location: internal/battencli/commitstatus.go:209
    summary: The new seam comment says a start transition commits nothing although startCore commits it.
    fix: Replace the statement that a start transition commits and pushes nothing with a statement that it commits without pushing.
    origin: lens:code-affected-set
---

### [BLOCKING] Fresh initialization bypasses the marker prerequisite

**Location:** internal/battencli/commitstatus.go:263

**Issue:** The push exception requires an existing marker, but a fresh run does not create the expected initial marker.
`battenPreRun` seeds status directly through `state.UpdateJSON` at `internal/battencli/arm.go:793`.
`seedAbsentStatus` names Worktree-Create and sets running at `internal/battencli/arm.go:819`.
Neither invokes the commit seam.
`stepLocked` therefore reads an already-started row at `internal/shedengine/run.go:240`.
Its start-write condition at line 323 is false, and Worktree-Create has no entry gate that could persist a transition.
The first seam call follows Worktree-Create's successful route to Seed-Child.
With no marker, line 263 selects the ordinary core, which pushes before Seed-Child acts.
This violates both the requested Seed-Child exception and `PATTERN-run-fsm`'s uninterrupted-start rule.
The new table supplies Worktree-Create/running manually at `internal/battencli/commitstatus_test.go:59`.
That sequence does not exercise fresh initialization.
The summary's claim that the fresh chain pushes at Worktree-Create also depends on this missing transition.

### [MEDIUM] Network-admission bypass lacks coverage

**Location:** internal/battencli/commitstatus_test.go:80

**Issue:** The new core deliberately omits `Admit` at `internal/battencli/commitstatus.go:236`.
Every case supplies a nil callback, so adding `Admit: deps.Admit` to that core would pass the assertions.
Such a regression would make local start commits wait on the network gate despite making no network call.
The shared-core admission tests cover the ordinary core, not batten's selection of dependencies for the start core.

### [LOW] Comment denies the commit it describes

**Location:** internal/battencli/commitstatus.go:209

**Issue:** “A start transition commits and pushes nothing” contradicts `startCore`'s `Commit: deps.Commit`.
The later sentence about carrying its commit confirms that only the push is omitted.

## Review evidence and limits

Reviewed the supplied committed diff and commit messages from `f660ac101279ab085ffbc815ebb3d1422d894030` to HEAD.
The board entry matches the decision record's goal.
The governing PATTERN constraint limits the exception to uninterrupted starts;
that constraint controls the broader wording of the goal for resumed and halted transitions.
Worktree-Create remains outside the new predicate, and Land and Worktree-Teardown retain the ordinary core.
Once a marker exists, the Run-Shed boundary, halt transitions and missing-marker fallback follow the documented branches.
The ordinary core retries an owed push before returning, and resumed passes perform catch-up before acting.
Later publication still uses the existing commit ancestry and `pairteardown.PushRun` takeover handling.
No new format or takeover protocol is introduced.

The mutation at line 225 changes warning emission only; the marker write still executes.
It does not establish a missing transition-result assertion.
The new table rows use literal expected counts, add no waits and preserve the old cases' exact assertions.
All lens findings were retained; their broader positive assessments do not override F1's initialization trace.

Attempted `lym gate test --tier 0 ./internal/battencli ./internal/statuscommit` in the foreground.
The gate could not write its holder record under the read-only `_bolt/.lym/gate` directory, so no tests ran.
This environmental limit is not a finding against the diff.
