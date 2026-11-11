---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    location: internal/boardengine/handles_test.go:229
    summary: TestWorkspaceFolderNames has no row for a project entry whose handle is empty, so the empty-handle branch is unpinned
    fix: In TestWorkspaceFolderNames, add to the board fixture a project entry with no handle (for example slug nohandle, kind project) and a task under it, and leave that task out of the want map of EntriesInAProjectTakeItsHandle, so a regression that names it with an empty handle fails.
    origin: lens:code-tests
---

### [LOW] Empty-handle project branch of WorkspaceFolderNames is unpinned

**Location:** internal/boardengine/handles_test.go:229
**Issue:** `WorkspaceFolderNames` leaves an entry out when its project exists but holds no handle (`handle != ""` in `internal/boardengine/handles.go`).
The `orphan` row covers only a project the board does not hold, which takes the same branch by a different path (a missing map key).
Dropping the `handle != ""` check would still pass every test, since `handles[task.Project]` is empty in both cases only for the missing-project row, and a project present with no handle would be named `: slug`.

## Rejected

- Lens code-tests items 11 and 12 (no test that the handle is read afresh on each regeneration, none that the namer runs before the lock): both pin a hypothetical later regression, a cache or a reordering, and the first is guaranteed by `WorkspaceFolderNames` holding no state; no behavior the diff adds goes uncovered.
- Lens code-correctness unverified caller of `RegenerateHubWorkspace` outside the lym root pre-run: every production caller (`ideengine/spawn.go`, `anchorchange.go`, `pawlwiring.go`) runs inside the `lym` binary, and `cmd/lym/main.go` is the only place that installs seams.
- Handler check of the focus questions: the hub file lists every pair at any depth plus the prime and Bolt without duplicates; a project-less pair, a project with no handle and an unreadable or absent board keep the bare slug; beam files keep slugs; the handle is read at each regeneration; the seam install keeps the existing pre-run behavior; docs and `_lym/landing/summary.md` bear out against the diff. Not defects.
