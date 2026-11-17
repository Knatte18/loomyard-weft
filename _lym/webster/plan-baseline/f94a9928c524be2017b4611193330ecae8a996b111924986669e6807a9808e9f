# Card 3 — Target snapshot seam for source and overlay scope

**Create:**
- `internal/lymdirs#ReviewSnapshotDirName`
- `internal/shedadapters#TargetSnapshot`
- `internal/shedadapters#TargetSnapshot.Take`
- `internal/shedadapters#TargetSnapshot.ID`
- `internal/shedadapters#TargetSnapshot.ChangedSince`
- `internal/shedadapters#TestTargetSnapshot`

**Extends:**
- `internal/shedadapters/doc.go`

**Uses:**
- `internal/lymdirs/doc.go`

**Intent:**
D5's target id and D4's changed-path list behind one seam both the review producer and the Bouncer hold.

`TargetSnapshot` fields: `Scope` (`burlerengine.FixScope`), `WorktreeRoot` (absolute), `Paths` (worktree-relative target paths, files or directories), `CacheDir` (absolute, the overlay copy root), `SourceTreeID func() (string, error)` and `SourceChanges func(fromID, toID string) ([]string, error)`.
The two closures are the Card 1 fabric verbs, wired later; the zero value of either makes source scope answer an error.

Overlay scope (`burlerengine.FixScopeOverlay`):
- `ID` walks every target path under `WorktreeRoot`, sorts the worktree-relative file paths and returns the hex sha256 over each path, a NUL, the file's bytes and a NUL, in order. A missing target path is skipped.
- `Take` returns `ID` and copies each file to `CacheDir/<id>/<relative path>`, skipping the copy when that directory already exists.
- `ChangedSince` compares `CacheDir/<fromID>` with the current target files and returns the sorted worktree-relative paths added, removed or with other bytes. A missing copy, or any read error, returns false.

Source scope (`burlerengine.FixScopeSource`): `ID` and `Take` return `SourceTreeID()`; `ChangedSince` returns `SourceChanges(fromID, current id)`.
Any error from either closure returns false.
A false `ChangedSince` is logged at Warn and never fails a round: the caller writes no changes file (D5).

`lymdirs.ReviewSnapshotDirName` names the cache directory under a worktree's `.lym`; the cache is never tracked and an absent one reads as empty (`PATTERN-lym-files`).
doc.go gains a paragraph on the seam.

`TestTargetSnapshot` is tier0, one table over a temp worktree.
Rows: an overlay id stays the same with no edit and changes when one target file is edited; a target path that does not exist is skipped, so the id equals that of the same set without it; overlay `ChangedSince` after `Take` lists exactly an edited, an added and a removed file and no unchanged one; a second `Take` over an existing `CacheDir/<id>` leaves the copy's bytes and modification times as the first wrote them; a deleted cache directory gives false; source scope returns the fake closures' id and their changed paths; a failing source closure gives false.

**ImpactSummary:** A new type and one lymdirs constant; nothing existing reads them yet.
