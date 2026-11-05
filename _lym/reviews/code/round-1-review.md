---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: decision
    origin: handler
    location: internal/beamcli/mergeparent.go:70
    summary: The merge task is a plain darn run whose writer runs lym fabric merge-in, but the entry says the run settles conflicts and gate failures with the merge fixer; the change description presents this as a design decision, not a recorded correction
    fix: Leave the code in place and defer this finding to the run's parent, which records the departure from the entry or refuses it with lym loom decision add; the fixer must not change the filed task's shape.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/beamcli/deps.go:184
    summary: The real Tasks, FileTask and CodeReviewCaps seams have no test; nothing fails if FileTask drops the SetStatus release (a draft task is never started by dobby) or CodeReviewCaps reads the wrong config or perch
    fix: Add a hub-fixture test at the tier the existing beamcli integration tests use (see dobbydeps_integration_test.go, which already builds hubLandDeps over a real hub) that calls hubLandDeps(prime).FileTask with merge-parent fields, then reads the board and asserts the task exists with a released status (nil, not draft), and asserts CodeReviewCaps returns the caps the hub's loom config sets for the code perch.
  - id: F3
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/beamcli/mergeparent.go:92
    summary: openMergeTask tests done-ness with the repeated literal "done" instead of the board's named constant, against PATTERN-named-constants
    fix: In openMergeTask replace `task.Status != nil && *task.Status == "done"` with `boardengine.IsDoneStatus(task.Status)`, and in mergeparent_test.go:20 replace `done := "done"` with a value built from `boardengine.StatusDone`.
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/beamcli/deps.go:187
    summary: When UpsertTask succeeds and SetStatus fails, a draft merge task is left on the board; the rerun then refuses with "stands open" and points at lym batten cancel --remove for a task that has no run
    fix: In the FileTask closure in deps.go, when SetStatus fails after UpsertTask succeeded, return an error that names the task's slug and says to remove it with `lym board` or retry the release, so the way forward the verb prints matches a task with no run; keep the two-step release.
---

### [MEDIUM] Merge fixer not used for conflicts and gate failures

**Location:** internal/beamcli/mergeparent.go:70
**Issue:** The amended entry says the run merges the parent into the beam and "settles conflicts and gate failures with the merge fixer". The diff files a plain darn task whose body tells the writer to run `lym fabric merge-in` and settle conflicts itself, with the darn verify gate's gate fixer for gate failures. `_lym/landing/summary.md` argues the merge fixer belongs to Parent-Align and Finalize and gives the reason, but the departure from the entry is not recorded with `lym loom decision add`. Per the rubric this is an unrecorded parent correction: it is not fixed by revert and is deferred to the parent.

### [MEDIUM] Real filing and budget seams are untested

**Location:** internal/beamcli/deps.go:184
**Issue:** `mergeparent_test.go` drives `mergeParent` over the `landWorld` fakes only, so `hubLandDeps`' `FileTask`, `Tasks` and `CodeReviewCaps` run in no test. The two-step `UpsertTask` then `SetStatus(slug, nil)` release is what makes dobby start the task, and nothing fails if it is dropped or reordered. `CodeReviewCaps` reading the prime's loom config for the code perch is likewise unpinned. This is a behavior the diff adds with no coverage.

### [LOW] Repeated "done" literal

**Location:** internal/beamcli/mergeparent.go:92
**Issue:** `boardengine.IsDoneStatus` and `StatusDone` exist at `internal/boardengine/store.go:454-458`; PATTERN-named-constants asks for one named constant referenced everywhere.

### [LOW] Partial filing leaves a misleading way forward

**Location:** internal/beamcli/deps.go:187
**Issue:** If the release write fails after the task was created, the board holds a draft merge task. A rerun finds it through `openMergeTask` and refuses with a `lym batten cancel --remove` way forward, which targets a run the task never had. The shape matches `fronttask`'s, and the case is narrow.

## Rejected

- Concurrent calls (a double click) can pick the same free slug and the upsert overwrites the first task (lens:code-correctness): narrow, a confirm prompt precedes the button, and every board filing by upsert shares it.
- `mergeTaskSlugPattern` also matches a hand-written `-merge-in` task in the project (lens:code-correctness NIT): the board has no marker field, and the change description states the choice.
- No test reaches the cobra wiring of `mergeParentCmd` (lens:code-tests LOW): it is a thin wrapper over `onHubLandDeps` and `finishVerb` shared with `land`; the sequence is tested through `mergeParent`.
- Failure paths of `ReadOwners`, `OriginRecord`, `Tasks`, `CodeReviewCaps` and `FileTask` have no rows (lens:code-tests LOW): plain error wraps with no branch to pin.
- No row combines a cut slug with a number above 1, and `title` is not asserted (lens:code-tests NIT): the cut and the numbering are each pinned by their own rows.
- The reed buttons use `{slug}` where the entry writes `{beam}`: `{beam}` resolves to the beam's directory, not its slug, so `{slug}` is the placeholder that makes `lym beam land` and `lym beam merge-parent` receive a slug; both templates carry the same rows and the prime group lists the other beam buttons without these two.
- The rubric stencil edit is not named in the entry: the entry's "its perch reviews the merge alone" licenses narrowing the review range, and the added lines drop, weaken or contradict no existing rule.
- Windows template parity not pinned by a test: both templates carry identical added rows in the diff.
- pawl untouched and no goal check added before landing: the diff touches no pawl file, matching the entry.
