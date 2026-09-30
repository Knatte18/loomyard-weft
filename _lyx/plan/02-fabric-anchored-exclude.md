# Card 2 — fabric-anchored-exclude

**Create:**
- `plan:internal/fabricengine#ExcludeAnchoredDir` -> `func ExcludeAnchoredDir(worktreeDir, anchorRel, dirName string) (excludePath string, changed bool, err error)`
- `plan:internal/fabricengine#PathTracked` -> `func PathTracked(worktreeDir, relPath string) (bool, error)`

**Edit:**
- `internal/fabricengine/gitexclude.go`
- `internal/fabricengine/gitexclude_integration_test.go`

**Uses:**
- `internal/fabricengine#mutateGitExclude`

**Intent:**
Add two exported helpers to `internal/fabricengine/gitexclude.go`, so the driven IDE path (card 3) keeps `.vscode/` out of git without a second locking scheme and without touching a tracked file.
The shared `info/exclude` is repo-wide and edited concurrently by `lyx fabric` verbs and other batten runs, which is why the write must go through the existing lock and atomic replace rather than a copy of them.

`ExcludeAnchoredDir` makes git ignore the directory `dirName` at the anchor subpath of the repository checked out at `worktreeDir`:
- It first asks git whether the path is already ignored, with `git check-ignore -q -- <anchorRel>/<dirName>/` run in `worktreeDir` through `gitexec.RunGit` (slash-separated, `anchorRel` `.` collapsing to the bare `<dirName>/`).
  Exit 0 means already ignored, by a tracked `.gitignore` or any other source, and returns `changed == false` with no write;
  exit 1 means not ignored and proceeds;
  any other exit code or a run failure is an error.
- Otherwise it calls `mutateGitExclude(worktreeDir, ...)` with a rewrite that appends the line `/<anchorRel>/<dirName>/` (slash-separated, `/<dirName>/` when `anchorRel` is `.`) unless that exact trimmed line is already present, and leaves every other line of the file as it was.
  The leading slash anchors the entry to this anchor's own directory, unlike a bare name, which gitignore matches at any depth.
  Unlike `excludePatternFor`'s junction patterns it keeps the trailing slash, since `.vscode` is a real directory rather than a junction git sees as a file.
- It returns `mutateGitExclude`'s resolved exclude path and changed flag, so the caller can log what it wrote.

`PathTracked` reports whether `relPath` (slash-separated, relative to `worktreeDir`) is in the index of the repository checked out at `worktreeDir`, via `git ls-files -- <relPath>` through `gitexec.Run`: non-empty output is tracked.
It is a read-only git query, exempt under the Fabric Git Invariant, placed here so every git call the IDE path makes stays in `fabricengine`.

Doc comments on both explain the anchoring, the already-ignored short-circuit, and the lock.
Neither function takes a `*Mutations` recorder: neither is a fabric CLI verb, matching `seedWeftArtifactExcludes`.

Tests in `gitexclude_integration_test.go` (integration-tagged, real repos as the file's existing tests build them), each asserting exact file content, not containment:
on a repo with a pre-existing custom exclude line, `ExcludeAnchoredDir(dir, ".", ".vscode")` leaves the file as the original lines plus exactly `/.vscode/`, and reports changed;
a second call reports unchanged and leaves the bytes identical;
with `anchorRel` `wts/some-task` the added line is exactly `/wts/some-task/.vscode/`, and `git check-ignore` confirms `wts/some-task/.vscode/tasks.json` is ignored while a root `.vscode/tasks.json` is not;
with a committed `.gitignore` already listing `.vscode/`, the exclude file is not created or changed and changed is false;
`PathTracked` is true for a committed file, false for an untracked one and false for an absent one.

**ImpactSummary:** Adds two exported functions to `fabricengine`; no existing symbol changes, and the only new caller arrives in card 3.
