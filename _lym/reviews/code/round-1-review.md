---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/fabricengine/add.go:338
    summary: Add deletes the leftover directory before its later pre-flight refusals, so a refused Add still destroys the path
    fix: "In add.go, keep the os.Stat check at line 338, but call removeIgnoredTaskPath only after the pre-flight refusals have passed. Move the delete and its logger.Info to just after the defer regenerateAfterVerb line (the marker comment reads that the pre-flight refusals have passed), before the createGitWorktree call. Until then a path that stands must be judged without deleting it, and a path that is not deletable must still return *ErrTaskPathTaken at the original position. Split removeIgnoredTaskPath into a read-only judgement that reports whether the directory is deletable and a delete step, or make the Add doc comment and the marker comment state the exception to 'refused before any mutation'. Prefer moving the delete."
    origin: lens:code-correctness
  - id: F2
    severity: MEDIUM
    class: design
    location: internal/fabricengine/add_taskpathleftover_integration_test.go:30
    summary: No row pins the claim that a file tracked in the checked-out repository, or a registered worktree, keeps the directory
    fix: "Add a row to the tests table of TestAdd_TaskPathLeftover whose leftover holds a file at a relative path that is tracked in the acting worktree's repository (commit it with gitkit.CommitFile under the same relative path, for example src/App/obj/tracked.cs, adding it past the ignore rule). Expect *ErrTaskPathTaken and every file kept. Do not add a registered-worktree row unless it can pass without a .git entry masking it; the .git row already covers a live worktree."
    origin: lens:code-tests
  - id: F3
    severity: LOW
    class: consistency
    location: internal/fabricengine/addadoptonly.go:73
    summary: ErrTaskPathTaken doc and message still say the directory already exists, though Add now deletes an ignored-only directory
    fix: In addadoptonly.go, reword the ErrTaskPathTaken type comment and its Error text to say the refusal is for a directory Add does not delete, meaning a registered worktree, a directory holding a .git entry, a directory holding a file git does not ignore, or a non-directory. Keep the way forward the consumers in battencli rely on.
    origin: lens:code-affected-set
  - id: F4
    severity: LOW
    class: consistency
    location: internal/fabricengine/add.go:175
    summary: Add can now return an error wrapping ErrStrayPathKept, and neither its doc comment nor doc.go says so
    fix: In the Add doc comment after the new exception sentence in add.go, add one line that a delete that fails returns an error wrapping ErrStrayPathKept and leaves the path on disk. Make the matching addition to the Add paragraph in doc.go beside the exception sentence, and to the ErrStrayPathKept sources list in doc.go near the Remove and RemoveStrayTaskPath bullets, and in the ErrStrayPathKept comment in remove.go.
    origin: lens:code-affected-set
  - id: F5
    severity: LOW
    class: design
    location: internal/fabricengine/destroy.go:1388
    summary: Lstat, List and walk failures in removeIgnoredTaskPath replace the ErrTaskPathTaken refusal with a bare error that callers turn into no guidance
    fix: In removeIgnoredTaskPath, treat a failed Lstat, List or filesUnderTaskPath like the check-ignore failure. Log a logger.Warn naming the path and cause, keep the path, and return false with a nil error, so Add returns *ErrTaskPathTaken as it did before. An Lstat that reports the path gone should return false with a nil error too, since Add then meets the same refusal it met before the change.
    origin: lens:code-correctness
  - id: F6
    severity: LOW
    class: design
    location: internal/fabricengine/destroy.go:1454
    summary: The check-ignore exit-1 branch and the delete-failure branch of removeIgnoredTaskPath have no test row
    fix: Add a row to TestAdd_TaskPathLeftover whose leftover holds only notes.txt, so git ignores none of the files and check-ignore exits 1; expect *ErrTaskPathTaken and the file kept. For the delete failure, add a case that makes the delete fail, through an export_test.go seam on Topology.leftover or a directory the delete cannot remove, and assert the error wraps ErrStrayPathKept and the path is still on disk.
    origin: lens:code-tests
---

### [MEDIUM] Delete runs before the later pre-flight refusals

**Location:** internal/fabricengine/add.go:338
**Issue:** The new branch deletes the leftover as soon as it meets it, but Add still has many refusals ahead: no remote, a missing weft repo, `ErrRecordsWorktreeNotPair`, `refuseAdoptLocalCheckedOut`, the detached HEAD check, `ErrNotLive`, the probe errors and the anchor refusal. A refused Add returns one of those errors with the directory already gone. The Add doc comment promises `ErrNotLive` and the adopt refusals "before any mutation", and the comment at the `regenerateAfterVerb` defer marks the point where the pre-flight refusals have passed. The delete sits before that point. The loss is only git-ignored output, but the contract and the retry behaviour change: a retry no longer meets the same state.

### [MEDIUM] Tracked-file and registered-worktree refusals are not pinned

**Location:** internal/fabricengine/add_taskpathleftover_integration_test.go:30
**Issue:** The commit message and the new `PATTERN-fabric-boundary` text claim the proof holds only when git ignores every file, and a tracked path is never ignored. No row puts a file at the leftover whose relative path is tracked in the acting worktree, so that part of the proof can break unseen. The registered-worktree refusal is masked by the `.git` refusal in any realistic row, so a tracked-file row is the practical pin.

### [LOW] ErrTaskPathTaken text no longer matches the behaviour

**Location:** internal/fabricengine/addadoptonly.go:73
**Issue:** The type comment and message describe the refusal as "the directory already exists". Add now deletes a directory holding only ignored files, so the refusal covers a narrower set than the text says.

### [LOW] ErrStrayPathKept now also leaves Add, undocumented

**Location:** internal/fabricengine/add.go:175
**Issue:** The new branch returns an error wrapping `ErrStrayPathKept` when the delete fails. The Add doc comment, the `doc.go` bullet and the sentinel's own comment name only `Remove` and `RemoveStrayTaskPath` as sources.

### [LOW] Operational failures replace the ErrTaskPathTaken refusal

**Location:** internal/fabricengine/destroy.go:1388
**Issue:** A failed `Lstat`, `List` or directory walk returns a wrapped error. Before the change, any existing path gave `*ErrTaskPathTaken`, which `battencli` turns into guidance for the parent. The check-ignore failure already keeps the path and logs, so the three other failures are inconsistent with it. An `Lstat` that finds the path gone after the `os.Stat` in Add reports an error where the old code would have refused or proceeded.

### [LOW] Two failure branches have no test row

**Location:** internal/fabricengine/destroy.go:1454
**Issue:** No row has every file unignored, so the exit-1 branch of `ignoresEveryPath` never runs, and no test makes the delete fail, so the `ErrStrayPathKept` wrap and its way-forward text are unpinned.

## Rejected

- Ignored user data such as `.env` is deleted without review (lens:code-correctness F3): the record's expected outcome is a reused slug whose path holds only ignored output, and `git worktree remove` already discards ignored files, so it is the intended proof, not a defect.
- check-ignore operands parsed as pathspecs, so a name starting with `:` is misread (lens:code-correctness F4): the misread path is reported as not ignored, so the directory is kept; no unsafe outcome.
- Symlinks, empty directories, a non-directory at the path and a `.git` entry at any depth (focus 2): each is handled safely and the delete still goes through `removeWorktreeLeftover` with `os.Root` containment and the `KindPathRemoved` record; not a defect.
- The ownership proof for paths outside the checked-out repository (focus 1): the leftover's relative paths are read as repo-root paths, so an anchored rule or a tracked path leaves the directory kept and a nested ignore file under the leftover is never consulted; every mismatch fails toward keeping the path. The `-z` parse and the exit-1 handling are correct.
- The commit message claims (focus 6): no worktree registers the path, no `.git` entry at any depth, and check-ignore ignores every file are each borne out by the code; the diff stays inside the entry's expected outcome.
- PATTERN and doc landing (focus 4): the third exception in `PATTERN-fabric-boundary` and `doc.go` matches what the code does; no `Pattern-Departure` trailer is needed, and `gitexec.RunStdin` is the permitted entry point.
- `TestAdd_TaskPathLeftover` duplicates an existing test (focus 5): it does not; the existing `ErrTaskPathTaken` test covers a file at the path under AdoptOnly, and the new one covers directories.
