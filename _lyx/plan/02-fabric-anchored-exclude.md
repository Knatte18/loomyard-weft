# Card 2 — fabric-anchored-exclude

**Create:**
- `internal/fabricengine#ExcludeAnchoredDir`

**Edit:**
- `internal/fabricengine/gitexclude.go`
- `internal/fabricengine/gitexclude_integration_test.go`

**Uses:**
- `internal/fabricengine#mutateGitExclude`

**Intent:**
Add one exported helper to `internal/fabricengine/gitexclude.go`, so the driven IDE path (card 4) keeps `.vscode/` out of git without a second locking scheme and without touching a tracked file.
The shared `info/exclude` is repo-wide and edited concurrently by `lyx fabric` verbs and other batten runs, which is why the write must go through the existing lock and atomic replace rather than a copy of them.

`ExcludeAnchoredDir` makes git ignore the directory `dirName` at the anchor subpath of the repository checked out at `worktreeDir`:
- It first asks git whether the path is already ignored, with `git check-ignore -q -- <anchorRel>/<dirName>/` run in `worktreeDir` through the checked `gitexec.Run` (slash-separated, `anchorRel` `.` collapsing to the bare `<dirName>/`).
  A nil error (exit 0) means already ignored, by a tracked `.gitignore`, the operator's global excludes or any other source, and returns `changed == false` with no write;
  a `*gitexec.GitError` (recovered with `errors.As`) whose `ExitCode` is 1 means not ignored and proceeds;
  any other error, including another exit code or git failing to run, is returned wrapped.
  Do not use the raw `gitexec.RunGit`: the gitexec Checked-Call Invariant pins `fabricengine` at two raw sites, and `cmd/lyx/checkedcall_test.go` fails on a third.
  git reports a directory holding tracked content as not ignored even when a rule covers it; the caller (card 4) never reaches this function in that case.
- Otherwise it calls `mutateGitExclude(worktreeDir, ...)` with a rewrite that appends the line `/<anchorRel>/<dirName>/` (slash-separated, `/<dirName>/` when `anchorRel` is `.`) unless that exact trimmed line is already present, and leaves every other line of the file as it was.
  The leading slash anchors the entry to this anchor's own directory, unlike a bare name, which gitignore matches at any depth.
  Unlike `excludePatternFor`'s junction patterns it keeps the trailing slash, since `.vscode` is a real directory rather than a junction git sees as a file.
- It returns `mutateGitExclude`'s resolved exclude path and changed flag, so the caller can log what it wrote.

The doc comment explains the anchoring, the already-ignored short-circuit, and the lock.
It takes no `*Mutations` recorder: it is not a fabric CLI verb, matching `seedWeftArtifactExcludes`.

Tests in `gitexclude_integration_test.go` (integration-tagged, real repos as the file's existing tests build them), each asserting exact file content, not containment:
- On a repo with a pre-existing custom exclude line, `ExcludeAnchoredDir(dir, ".", ".vscode")` leaves the file as the original lines plus exactly `/.vscode/`, and reports changed;
  a second call reports unchanged and leaves the bytes identical.
- With `anchorRel` `wts/some-task` the added line is exactly `/wts/some-task/.vscode/`, and `git check-ignore` confirms `wts/some-task/.vscode/tasks.json` is ignored while a root `.vscode/tasks.json` is not.
- With a committed `.gitignore` already listing `.vscode/`, the exclude file is not created or changed and changed is false.
- The write holds the exclude lock: the test takes `lock.AcquireWriteLock` on `gitExcludeLockFileName` beside the resolved exclude path, starts `ExcludeAnchoredDir` in a goroutine, and asserts that after a short wait (well under a second) the call has not returned and the file is unchanged.
  It then releases the lock and asserts the call returns changed with the line added and the pre-existing custom line intact.
  `flock` conflicts between two opens in one process, so this holds in-process.

**ImpactSummary:** Adds one exported function to `fabricengine`; no existing symbol changes, and the only new caller arrives in card 4.
