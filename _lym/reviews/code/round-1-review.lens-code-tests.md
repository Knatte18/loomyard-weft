```yaml
lens: code-tests
outcome: reported
```

**Per-behaviour list (code-tests lens)**

- Ignored-only directory deleted, pair made, `KindPathRemoved` "recursive" recorded: `TestAdd_TaskPathLeftover/OnlyIgnoredOutputIsDeleted`.
- Empty directory deleted (the vacuous `len(rels)==0` branch): `EmptyDirectoryIsDeleted`.
- File git does not ignore keeps the directory (the `!ignored[rel]` branch, a mixed set where check-ignore exits 0): `FileGitDoesNotIgnoreKeepsIt`. It also asserts the files stay and no removal is recorded.
- `.git` entry at depth keeps the directory (`filesUnderTaskPath` `holdsRepository`): `GitEntryKeepsIt` (`src/App/obj/.git`).
- Non-directory at the path stays `ErrTaskPathTaken`: `add_adoptonly_integration_test.go:365` "refuses a file at the task worktree path". It pins the `!info.IsDir()` branch, though only via the AdoptOnly path. This is no duplicate of the new test.
- Registered linked worktree at the path is refused (`listsLinkedWorktree`): none. A live worktree has a `.git` entry and tracked files, so the other refusals mask a break here.
- Tracked path never counts as ignored (the proof's "tracked in the checked-out repository" case): none. No row puts a file at the task path whose relative path is tracked in `l`'s repository.
- `check-ignore` exit 1, when git ignores none of the paths: none. Every non-ignored row mixes in an ignored file, so git exits 0. Breaking the exit-1 branch changes only the warning log, so this is hard to pin.
- `check-ignore` failing with another exit code or exec error leaves the path kept and logged: none.
- Delete failure returns an error wrapping `ErrStrayPathKept`: none. The `leftoverSeam` offers `remove` injection, but `Add` passes `t.leftover` and no test uses it here.
- `Lstat` or `List` failure returns an error: none.
- Symlink entries under the leftover (`WalkDir` does not follow them): none.
- Rollback or later-step failure after the delete: none.
- `-z` parsing with unusual file names: none. The ordinary names in the rows do pin the NUL-split.

**Findings**

1. MEDIUM, design. Location `internal/fabricengine/add_taskpathleftover_integration_test.go:30-35` (the rows table).
   - Summary: no row pins the refusal of a registered linked worktree at the path, or of a file tracked in the checked-out repository.
   - Why it matters: both are the ownership claims the new `PATTERN-fabric-boundary` text and the commit message make.
   - Evidence: `destroy.go` `removeIgnoredTaskPath` (the `listsLinkedWorktree` branch) and the `ignoresEveryPath` doc, "A tracked path is never ignored". Neither is exercised by any `TestAdd_TaskPathLeftover` row.
   - Suggested row: a leftover directory whose relative file paths are tracked in `l`'s repository, for example a committed `src/App/obj/x.cs` force-added past the ignore. Expect `ErrTaskPathTaken` and the files kept.

2. LOW, design. Location `internal/fabricengine/destroy.go` `removeIgnoredTaskPath`, the failure branches (`check-ignore` failure, delete failure, `List` or `Lstat` failure).
   - Summary: no test reaches them.
   - Evidence: no row in the new test, and no use of `leftoverSeam.remove` from `Add`. The delete-failure branch carries the user-facing way-forward text and the `ErrStrayPathKept` wrap, which is untested.
   - Suggested coverage: an `export_test.go` seam, or a row whose directory cannot be deleted.

3. LOW, design. Location `destroy.go` `ignoresEveryPath`, the exit-1 branch.
   - Summary: no row has every file unignored, so git exits 1.
   - Suggested row: a directory holding only `notes.txt`. Breaking the branch changes only the log, so it is a coverage hygiene gap.

Test economy: `TestAdd_TaskPathLeftover` does not duplicate the existing `ErrTaskPathTaken` test at `add_adoptonly_integration_test.go:365`. That one covers a file at the path under AdoptOnly, and the new one covers directories. The redundancy report lists the new test as "not judged, unclassifiable call", so there is no finding on that ground.

Tool calls:
1. `sed` on `add_adoptonly_integration_test.go:345-395`: my context lacked the existing `ErrTaskPathTaken` test, which I read to judge overlap and the non-directory case.
