```yaml
lens: code-affected-set
outcome: reported
```

Task: lens code-affected-set. For each change the diff makes, here is what depends on it and whether the diff covers it.

**Per-change set**

1. **`dotgit.ListWorktrees`, `Worktree` and `ErrRefStorage` (new exports).**
   - Set: `internal/dotgit/doc.go`, `pattern/PATTERN-leaf-packages.md` and `PATTERN.md`.
   - `doc.go` and the pattern background carry the new text. They list the worktree entries, the refs and the four config keys, and state the named refusal. The declared text matches the `PATTERN.md` index line at line 11, which keeps its wording, so no `PATTERN.md` edit is needed. Covered.
   - `PATTERN-gitrepo-client-boundary.md` line 7 says `internal/gitrepo` makes local reads via `readGoGit` and `dotgit`. It does not name `fabricengine` or `websterengine` as `dotgit` readers.
   - `PATTERN-cwd-resolution.md` line 16 describes `lymcwd`'s imports, which are unaffected.
   - The decision record gives the first of these reads to `dotgit`, so this is not a finding. Covered.
2. **`dotgit` `TestMain` moved to the external package `dotgit_test` and calls `gitkit.HermeticGitEnv`.**
   - Set: `testmain_test.go`, the in-package test files `worktrees_test.go` and the existing tests, and `worktrees_git_test.go`.
   - One `TestMain` serves both packages in a single test binary, so the existing in-package tests still run under it. The new git-spawning test reaches the hermetic env. Covered.
3. **`fabricengine.List` behavior: no git process, new error text, and the missing-directory error is `fs.ErrNotExist` rather than a git exit error.**
   - Set: 30 or more `List` callers, from my grep of `internal/`.
   - Of the callers I checked, none asserts git's old error text. Their wrapping messages ("list worktrees") are unchanged. The `junction.go` and `scratchlink.go` fallbacks key on any error, so they still fire. `TestList_Errors` pins the new error text. Covered.
4. **Stale comments that say these callers run `git worktree list`.** These are a finding.
   - `internal/fabricengine/commit_integration_test.go:288` says a missing anchor subdirectory "makes `git worktree list` fail". It now fails inside `dotgit` (`requireDir`). The test still passes because the error is wrapped as "resolve main worktree".
   - `internal/fabricengine/pairbranch_integration_test.go:295` ("the worktree list RecordsRepoRoot resolves through cannot run") is still accurate.
   - `internal/fabriccli/hubmutation_integration_test.go:346`, `internal/fabricengine/destructivegaps_integration_test.go:788` and `internal/fabricengine/reconcile.go:188` all say "reads `git worktree list` once". `internal/fabricengine/doc.go:1787` says "in `git worktree list` order", which remains true because the order is preserved.
   - `internal/fabricengine/doc.go:1262` was updated in the diff. The comments in the `fabriccli` and `fabricengine` tests were not.
5. **`websterengine.otherWorktrees` now reads through `dotgit`.**
   - Set: `internal/websterengine/gitwrap.go`, `internal/websterengine/doc.go` (updated), the test move and the `Geometry.Git` fake.
   - The doc says `Geometry.Git` carries the "linked worktrees" fake, but `otherWorktrees` never used it, so nothing changes there.
   - The `TestRepositoryProbes` `testtiming:keep` text was updated and the sub-test was moved to `TestOtherWorktrees`. Covered.
6. **`gitkit.SkipBelowGitVersion` (new helper).**
   - Set: its three call sites, in the `dotgit`, `fabricengine` and `websterengine` tests.
   - The helper sits in `internal/gitkit/query.go`, which PATTERN says holds test git plumbing, so it is in the right place. It is not directly tested. Whether that is a gap is the tests lens's call.
   - `compareDottedVersions` duplicates what `internal/toolfloor` holds ("the git and tmux version floor and the version-line parser"). PATTERN-leaf-packages lists it as the version-line parser. `gitkit` may not import `toolfloor`, since its import set is closed. So this is not a violation, but see finding F1.

**Findings**

- **F1, LOW, consistency.** `internal/gitkit/query.go:125-157`: `SkipBelowGitVersion`, `compareDottedVersions` and `dottedComponent` add a second git version-line parser and comparer.
  - The repo already has `internal/toolfloor`, the version-line parser, which spawns nothing.
  - `gitkit`'s closed import set forbids importing it (PATTERN-leaf-packages, `internal/gitkit`). I did not read `toolfloor`'s API, so check whether it exports a parse and compare that could serve here before keeping the duplicate. If it does not, this is not a defect.
- **F2, NIT, consistency.** Stale comments after the change, all saying the read runs `git worktree list` when `List` now reads files: `internal/fabricengine/commit_integration_test.go:288`, `internal/fabricengine/reconcile.go:188`, `internal/fabriccli/hubmutation_integration_test.go:346` and `internal/fabricengine/destructivegaps_integration_test.go:788`.
  - Fix: reword each to "reads the worktree list once", as `internal/fabricengine/doc.go:1262` already does.
  - The `commit_integration_test.go:288` comment is wrong about where the failure arises; the others are only loosely worded.

I found no "Declared PATTERN changes landed" gap. The leaf-packages background carries the worktree entries, the refs, the config keys and the named refusal.

I found no commit message that departs from the diff. Neither commit has a `Pattern-Departure:` trailer, and none is needed.

**Tool calls**
1. `grep` for `worktree list` text across `internal/`: my context did not hold the comments and error-text assertions that name it.
2. `grep` of `PATTERN.md` and `pattern/` for `dotgit` plus `sed` of `dotgit.go`: to confirm the index line and how `requireDir` and `Read` behave.
3. `grep` of `fabricengine/doc.go`, plus the `commit_integration_test.go` and `pairbranch_integration_test.go` spans: to judge the stale comments and any error-text assertions.
