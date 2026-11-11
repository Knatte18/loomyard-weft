---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    location: internal/battencli/dobbyreads.go:165
    summary: no test pins DobbyRuns setting EntryEnded and PairPresent, the wiring that frees the slot in production
    fix: Add a done-entry-with-pair-present case and an entry-gone-from-the-board case to an existing DobbyRuns reading test, asserting RunView.EntryEnded and RunView.PairPresent. Add no new test function.
    origin: lens:code-tests
  - id: F2
    severity: LOW
    class: design
    location: internal/dobbyengine/loop.go:563
    summary: no test pins the binary check being held while a teardown is in flight, nor a nil TearDownEnded tearing nothing down
    fix: In TestTearDownEnded in internal/dobbyengine/ended_test.go, add a row that counts CheckBinary calls while a never-delivering teardown is pending and expects none. Add a row with TearDownEnded nil expecting no teardown call and no busy mark.
    origin: lens:code-tests
  - id: F3
    severity: LOW
    class: design
    location: internal/dobbyengine/ended.go:99
    summary: the one-hour retry ceiling and the forgetting of a wait once a run is no longer stranded have no assertion
    fix: In the failing-teardown subtest of TestTearDownEnded, supply enough failing results that the doubled wait reaches endedTeardownCeiling and assert it stops there. In the removed-pair subtest, assert that endedRetries holds no entry for the slug after the PairPresent=false ticks.
    origin: lens:code-tests
---

### [LOW] No test pins the DobbyRuns wiring of EntryEnded and PairPresent

**Location:** internal/battencli/dobbyreads.go:165
**Issue:** `TestEntryEnded` pins only the `entryEnded` helper.
The assignments of `EntryEnded` from the local board and `PairPresent` from the task worktree path in `DobbyRuns` are what frees the slot and marks a run stranded in production, and no row fails if either is dropped or miswired.

### [LOW] Binary-check hold and nil seam are unpinned

**Location:** internal/dobbyengine/loop.go:563
**Issue:** The `d.teardown == nil` guard around `CheckBinary` and the no-op of a nil `Deps.TearDownEnded` are documented behaviors that no test row exercises.

### [LOW] Retry ceiling and wait forgetting are unpinned

**Location:** internal/dobbyengine/ended.go:99
**Issue:** The failing-teardown subtest stops at the second doubling, so `endedTeardownCeiling` is never reached.
The deletion of a wait for a run that is no longer stranded passes the removed-pair subtest whether or not it runs.

## Rejected

- Lens correctness F1 (a run missing from a stale local board frees its slot): the Goal says "done or gone", and the local board is the one dobby already reads for every start and `NoSlot` decision; the teardown itself is judged from origin, so no pair is removed on the local board's word.
- Lens correctness F2 (a "pair removed" line repeating while the path stays on disk): it needs the reconcile to report a removal that the pair teardown's contract rules out, so it is speculative.
- Lens tests F-E (only single-slug rows): the per-slug logic is one loop body already exercised by every row, so a two-slug row adds no behavior.
- Lens affected-set: no findings; docs, `RunView` constructors, `BuildView` and PATTERN entries were checked and found consistent.
- Scope, PATTERN fit and the focus questions (teardown matches `lym cleanup --apply` through the same `ReconcileOrigin` hub scope and `pairteardown` seams; both Idea bullets met; nothing beyond them changed) raised no further finding.
