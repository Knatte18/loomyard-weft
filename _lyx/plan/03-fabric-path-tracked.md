# Card 3 — fabric-path-tracked

**Create:**
- `internal/fabricengine#PathTracked`
- `internal/fabricengine/pathtracked.go`
- `internal/fabricengine/pathtracked_integration_test.go`

**Intent:**
Add `PathTracked` in a new file, `internal/fabricengine/pathtracked.go`, so the driven IDE path (card 4) can ask whether the repo tracks its `.vscode/tasks.json` without a raw git call outside `fabricengine`.
It lives in its own file rather than `gitexclude.go`, whose header scopes that file to `info/exclude` mutation.

`PathTracked` reports whether `relPath` (slash-separated, relative to `worktreeDir`) is in the index of the repository checked out at `worktreeDir`, via `git ls-files -- <relPath>` through the checked `gitexec.Run`: non-empty output is tracked, empty output is not, and a git error is returned wrapped.
It is a read-only git query, exempt under the Fabric Git Invariant, placed in `fabricengine` so every git call the IDE path makes stays there.
Its doc comment states the index-only meaning: an untracked file on disk, and an absent path, both report false.
A file header comment names the file's one job.

Tests in the new integration-tagged file, on a real repo built the way `gitexclude_integration_test.go`'s `newGitRepoForExcludeTest` builds one (the package's `TestMain` already runs `gitkit.HermeticGitEnv()`):
`PathTracked` is true for a committed file, false for an untracked file present on disk, and false for an absent path;
and true for a committed file under a subdirectory, asked with its slash-separated path.
