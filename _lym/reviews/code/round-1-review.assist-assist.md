---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: internal/shuttleparity/seats.go:126
    summary: The shared seat holder repeats a rename-sensitive symbol path instead of declaring it once.
    fix: Declare one named constant for internal/landingshed#soloSession.table in seats.go and reference it in all three changed Holders entries.
    origin: lens:code-affected-set
  - id: F2
    severity: MEDIUM
    class: design
    location: internal/landingshed/finalize.go:339
    summary: A parent-holds result cached before reconciliation can bypass landing blockers for newly fetched work.
    fix: Preserve the initial parent-work read for ownerChecks, but refresh the predicate after reconciliation before landCheck and syncParent consume it. Preserve each caller's former failure handling, or restore their individual reads. Add a regression row to the existing land-check test where reconciliation advances the task from held work to unheld work while a blocker remains.
    origin: lens:code-correctness
  - id: F3
    severity: LOW
    class: scope
    location: internal/landingshed/finalize_pr.go:133
    summary: Sharing the description reader changes Finalize's diagnostic despite the unchanged-behaviour requirement.
    fix: Preserve the parse change description error prefix for Finalize and the read the change description prefix for PR-Review, by leaving parsing and its error wrap with each caller or supplying the caller-specific wording to the shared reader.
    origin: lens:code-scope
  - id: F4
    severity: LOW
    class: scope
    location: internal/landingshed/parentalign.go:285
    summary: Parent-Align now mutates migrated attempt records before a failed row-entry read.
    fix: Restore ParentAlign.readAttempts reading RowEntry and returning its error before either moveScratchAttempts call. Pass the resolved entry into shared record-reading code, keeping Finalize's existing ordering. Cover the failure ordering with a row in the existing migration behaviour test.
    origin: lens:code-scope
  - id: F5
    severity: NIT
    class: consistency
    location: _lym/landing/summary.md:22
    summary: The reported final production comment count does not match the reviewed tree.
    fix: Refresh the final comment count using the stated codestats command; the reviewed tree reports 2573 production comment lines, not 2570. Refresh affected measurements again after fixes.
    origin: handler
---

### [BLOCKING] Declare the holder path once

**Location:** internal/shuttleparity/seats.go:126

**Issue:** The three changed catalog entries repeat `internal/landingshed#soloSession.table`.
`pattern/PATTERN-code-conventions.md` requires a name or path that can go stale to have one named constant.
The review directive explicitly makes a listed constraint violation BLOCKING, regardless of its size.

### [MEDIUM] Refresh parent-work state after reconciliation

**Location:** internal/landingshed/finalize.go:339

**Issue:** `held` is read before `ownerChecks`, which reconciles even when it receives true.
The production seam calls a moving run-scope reconcile.
`internal/fabricengine/originreconcilemoves.go` can fast-forward the task checkout during that reconcile.
A local beam tip already held by its parent can therefore become a newer, unheld tip before `landCheck` runs.
Previously that check reread the predicate and consulted `LandBlockers`.
Now the stale true value skips the blockers and parent synchronization.
The parent lock does not prevent the task checkout from moving.
This violates the retained Finalize rule that an open member or forked pair blocks a beam's landing.

### [LOW] Preserve Finalize's error wording

**Location:** internal/landingshed/finalize_pr.go:133

**Issue:** A missing or malformed description changes from `parse change description` to `read the change description`.
The landing summary acknowledges this change, but the board entry requires unchanged behaviour and carries no amendment.
Sharing the reader does not require changing the caller's diagnostic.

### [LOW] Preserve failure ordering before migration

**Location:** internal/landingshed/parentalign.go:285

**Issue:** Parent-Align formerly returned a failed `RowEntry` read before migrating any scratch attempt record.
It now migrates first, then reads the entry through `readRowAttempts`.
With an existing scratch record and a failed entry read, it writes the durable record and removes the scratch copy.
The former path changed neither location.
The task excludes migration changes and requires unchanged behaviour.
The test lens independently found that existing migration rows do not cover this failure ordering.
Its finding is consolidated here rather than counted again as a separate coverage defect.

### [NIT] Refresh the final comment measurement

**Location:** _lym/landing/summary.md:22

**Issue:** Running `go run ./tools/codestats -root internal/landingshed -json` on the reviewed tree reports 2,573 comments.
The summary reports 2,570.
The reported final production code count of 5,131 and test code count of 11,003 match the measurement.

## Review coverage

The range is `bd00aab14fe7088f78342897f3837a08343aa590..HEAD`.
The board entry and decision record agree and contain no added amendment.
The fixer lock release, front-of-queue retry and baseline adjustment are retained with the reasons the scope permits.
The one changed test line adapts its return-value access and retains its assertions.
The parity catalog update is a required caller change; F1 concerns its representation.
The shared report parsers retain their distinct verdict and required-section validation.

Package vet passed for `internal/landingshed` and `internal/shuttleparity`.
No test run was performed by this reviewer; existing tests were inspected by the test lens.
The stale-state finding rests on the production reconciliation path, not a new executable fixture.
