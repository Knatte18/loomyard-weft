---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D6
    summary: D6 retires the portal_restored action, but that action also reports a launcher-only restore, and D6 leaves what such a restore reports undecided
    fix: In decision-record.md D6, after the sentence listing what goes, add that Reconcile's launcher restore stays and that a pair whose launcher directory it restored reports a new `launchers_restored` action in place of `portal_restored`, so a consumer keying off Action still sees that convergence altered the pair; add an acceptance criterion that a pair whose launcher directory was deleted reports `launchers_restored` after `lyx fabric reconcile`.
  - id: F2
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md D2
    summary: Regeneration now overwrites the file on every pair add or remove, silently dropping any edit VS Code or the operator made to it, and VS Code's own writes are not serialized by the D5 lock
    fix: In decision-record.md D2, add a sentence stating that the file is owned by lyx, that every regeneration overwrites edits made to it in VS Code or by hand, and that workspace settings belong in the prime's `.vscode/settings.json`; add an Open risks bullet that a folder or setting added through VS Code's workspace UI is lost at the next regenerating verb.
  - id: F3
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md D3
    summary: D3 and the failing-regeneration criterion say a failure before the write writes nothing, but D5 creates `_board/.lyx` and the lock file before the listing
    fix: In decision-record.md D3, change "On any failure before the write it writes nothing" to "On any failure before the write it writes no workspace file", and in the acceptance criterion on failing regenerations change "writes nothing" to "writes no workspace file".
---

### [MEDIUM] Retiring `portal_restored` leaves launcher-only restores unreported

**Location:** `_lyx/discussion/decision-record.md` D6

**Issue:** D6 removes "Reconcile's portal restore and its `portal_restored` action".
In `internal/fabricengine/reconcile.go`, `restorePortalAndLaunchers` restores both the portal link and the launcher directory, and `ReconcileActionPortalRestored` is set when either was restored; its doc comment says the action exists so "a consumer keying off Action must see that convergence altered the pair".
`_launchers` and its scripts are out of scope and stay, so the launcher restore stays too, but with the action gone a pair whose launcher directory was rebuilt reports `already_healthy`, the exact defect the action was added to close.
D6 neither keeps, renames nor deliberately drops that signal, so the plan writer has to invent the outcome.

### [LOW] The workspace file overwrites VS Code's own edits on every pair change

**Location:** `_lyx/discussion/decision-record.md` D2

**Issue:** VS Code writes a `.code-workspace` file itself when the operator adds a folder or changes a workspace-level setting through its UI.
Today the file is rewritten only by `lyx ide spawn <prime>`; under D4 it is rewritten by every `Add`, `Remove`, `Checkout` and `Prune`, which batten runs throughout a session.
The record never says the file is lyx-owned or that such edits are lost, and VS Code's write is not under the D5 lock, so it can also interleave with D5's truncating in-place write.
The loss is bounded (the operator can move settings into the prime's `.vscode/settings.json`), but the operator is never told.

### [NIT] "Writes nothing" is contradicted by the lock's own directory and file

**Location:** `_lyx/discussion/decision-record.md` D3

**Issue:** D5 has the regeneration create `_board/.lyx` when it is absent and take `hub-workspace.lock` there before the listing, so a lock timeout or an unreadable-settings failure has already written a directory or a lock file.
D3's "writes nothing" and the matching acceptance criterion mean the workspace file only, and say so less precisely than the rest of the record.

## Focus outcomes

- Focus 1, D3, D4 and the criteria against D5's in-place write: no defect.
  `WriteHubWorkspace` in `internal/fabricengine/hubworkspace_write.go` calls `os.Root.WriteFile`, which truncates before writing.
  A failed or killed write leaves a truncated or partial file; D3 says so, D4's Bound names the next regeneration as the repair, since the bytes differ and the unchanged-file shortcut does not apply, and the criteria pin both the failure and the rewrite.
  A killed process logs no Warn, so the partial file persists until the next regenerating verb or prime spawn, which is exactly what D4's Bound says.
- Focus 2, D5's lock: no defect beyond F3.
  The path `<hub>/_board/.lyx/hub-workspace.lock`, the 5-second wait and the no-`_board` behaviour (no lock, no directory, no Bolt) are all stated.
  `lock.AcquireWriteLockWithin` creates no directories, and D5 has the regeneration create `_board/.lyx` itself, as `hubreconcile`'s `walk.go` does with `os.MkdirAll` before its own `AcquireWriteLockWithin`.
  `_board/.lyx/` is excluded in the weft repo's `.git/info/exclude`, so the lock file stays untracked.
- Focus 3, new bounds in D2, D4, D6 and D7: no defect.
  D7 keeps a directory at the retired workspace path with a detail, excludes it from the Warn, and has a criterion for it.
  A prime branch change through `Topology.Checkout`, rollback included, renames the first folder from the branch D2 reads; D4 states this and that it restarts extensions.
  `ReconcileResult` today carries only `Pairs`, `WarpBinding` and `WarpBindingDetail`; D7 adds `HubDetails []string` and has `runReconcile`'s hand-built envelope emit it, so no Decision relies on a field that does not exist.
