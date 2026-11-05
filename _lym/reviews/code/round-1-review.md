---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: decision
    origin: handler
    location: internal/testkit/tmuxkit/tmuxkit.go:87-105
    summary: "The guard watches only a new .git, not any new entry under /tmp or the gate's temp root, and the leaker the entry asks to find is still unfound; neither departure is recorded."
    fix: "Leave the code in place; do not revert the narrowing to .git. Defer this finding to the run's parent, who records or refuses the two departures from the board entry's Idea with lym loom decision add. The departures are the .git-only check and the open source hunt. Recording them is outside the fixer's write surface."
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/testkit/tmuxkit/tmuxkit.go:87-105
    summary: "Main's wiring of the check (snapshot before m.Run, check after it, exit code propagated) has no test, and the Linux and Windows paths each repeat it."
    fix: "In tmuxkit.go, extract the shared sequence into one function that takes the run function as a parameter. It does the snapshot, runs the function, calls checkNewGitEntries and returns the code. Main passes m.Run on both the Windows and Linux paths. In tmuxkit_test.go add a row-driven test of that function with a fake run that creates a .git in a t.TempDir directory, asserting exit 1, and a fake run that creates none, asserting the code is unchanged."
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/testkit/tmuxkit/tmuxkit.go:137-141
    summary: "The fallback branch always says the entry is not an empty directory, though it is also reached when os.Remove fails on an empty directory or a concurrent binary already removed it."
    fix: "In checkNewGitEntries keep the Lstat and IsDir test but report each outcome separately. Say already removed when Lstat or os.Remove returns fs.ErrNotExist. Print the os.Remove error for an empty directory that cannot be removed. Keep the remove it by hand text for a non-directory entry or a non-empty directory. Add table rows only where a branch is reachable without root."
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/testkit/tmuxkit/tmuxkit_test.go:124-160
    summary: "Every TestCheckNewGitEntries row seeds only the second watched directory, so a regression that watches only one of the two directories is not caught."
    fix: "Add a seed target to the table, for example a field naming which of the two directories holds the entry, and add a row that places a new empty .git in the first directory, expecting exit 1, the path named and the directory removed."
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/testkit/tmuxkit/tmuxkit_test.go:150-160
    summary: "No row seeds a symlink named .git, so the Lstat choice that keeps a link instead of following or removing it is unpinned."
    fix: "Add a row NewSymlinkFailsAndIsKept whose seed calls os.Symlink to a path under t.TempDir, expecting exit 1, the remove it by hand text and the link kept. Skip the row on Windows in the loop before t.Run, as PATTERN-test-speed prescribes, since links may need a privilege there."
  - id: F6
    severity: NIT
    class: design
    origin: lens:code-correctness
    location: internal/testkit/tmuxkit/tmuxkit.go:132
    summary: "Ranging over the presentGitEntries map orders the messages randomly when both shared directories hold a new .git."
    fix: "Loop over dirs in order in checkNewGitEntries and Lstat each dir's .git there, instead of ranging over the map."
---

### [MEDIUM] Narrowed guard and open source hunt are not recorded
**Location:** `internal/testkit/tmuxkit/tmuxkit.go:87-105`
**Issue:** The board entry asks for two things: find the test or code that creates `.git` under the shared temp directory, and add a guard that fails a run leaving a new entry directly under `/tmp` or the gate's temp root.
The diff adds a guard for a new `.git` only, and `_lym/landing/summary.md` says the leaker was not found.
The narrowing is reasoned: other programs and concurrent binaries legitimately add entries to both directories.
It still departs from the entry, and no `Added after Discussion` entry records it.
The guard does name the failing package the next time the leak happens, so the open hunt is a defensible outcome; the decision belongs to the parent.

### [MEDIUM] Main's wiring has no test
**Location:** `internal/testkit/tmuxkit/tmuxkit.go:87-105`
**Issue:** The helper tests pin `presentGitEntries`, `checkNewGitEntries` and `sharedTempDirs`.
Nothing fails if `Main` drops the snapshot, takes it after `m.Run`, or loses the check on one OS, and the Windows and Linux branches repeat the sequence.
This wiring is the behavior the change exists for.

### [LOW] The fallback message misreports two cases
**Location:** `internal/testkit/tmuxkit/tmuxkit.go:137-141`
**Issue:** An empty directory that `os.Remove` cannot remove, and an entry a concurrent binary has already removed, both print that the entry is not an empty directory.
The exit code is correctly 1, so nothing passes falsely; the message sends the reader to remove something wrongly described or already gone.

### [LOW] No row puts the new entry in the first watched directory
**Location:** `internal/testkit/tmuxkit/tmuxkit_test.go:124-160`
**Issue:** All rows seed `dirs[1]`.
A change that scanned only the second directory would pass.

### [LOW] No symlink row
**Location:** `internal/testkit/tmuxkit/tmuxkit_test.go:150-160`
**Issue:** The `Lstat` and `IsDir` branch that keeps a link is covered only by the gitfile row, which is a regular file.

### [NIT] Message order depends on map iteration
**Location:** `internal/testkit/tmuxkit/tmuxkit.go:132`
**Issue:** With a new `.git` in both directories, the two reports can come in either order.

## Rejected

- `lens:code-affected-set` LOW: the docs say the system temp directory is above the lym root without stating it is `os.TempDir()` at startup. The fork itself found no defect, and the docs hold in the inherited-run-directory case.
- `lens:code-affected-set` NIT: `tmuxkit.go:109` repeats the name `gitEntryName` that `internal/hubgeom/hubgeom.go:51` declares. The packages are separate and a test kit imports almost nothing, so no shared constant is warranted.
- `lens:code-tests`: a missing row for an incoming exit code of 1. Preserving it is a single `return code`, and a row would only restate that.
- Race of an empty-directory `os.Remove` with another concurrent binary: it cannot remove a populated directory. A binary that snapshots after the `.git` appeared is not failed, by design. The documented bound states that concurrent packages fail for the same entry.
- Ordering and early return in `Main`: the snapshot follows `lymtemp.Dir()`, so it sees the final directories. The early return on a socket path failure runs no tests, so it needs no check.
- `sharedTempDirs` under an inherited run directory, a symlinked temp directory or Windows: `resolveRootPath` keeps the root beside the parent's, and `filepath.Dir(root)` is the system temp directory. The two directories never coincide.
- Commit message and summary claims: the cases the summary lists for `TestCheckNewGitEntries` match the table, and the package doc and `PATTERN-test-isolation.md` bullets match what `Main` does (exit 1, naming, removal of an empty directory, the concurrency bound).
