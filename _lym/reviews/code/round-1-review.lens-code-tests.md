```yaml
lens: code-tests
outcome: reported
```

**Lens: code-tests.** For each behaviour the diff adds or changes, I looked for the test row that fails when that behaviour breaks.

**Which test pins each behaviour**
1. **Two-repository hub, cherry-pick:** with a weft cherry-pick in progress, `MergeStateOperation` and `MergeStateActive` answer `""` and false. Pinned by `TestMergeStateActive/weft cherry-pick in progress reports false` (`internal/fabricengine/mergestateactive_integration_test.go:97-112`). It would fail against the pre-fix code, which answered `"cherry-pick"`.
2. **Two-repository hub, status commit during a records cherry-pick:** the transition commits. Pinned by `TestCommitStatusSeam_Real/records cherry-pick in progress commits the transition` (`internal/loomcli/wiring_commitstatus_integration_test.go:400-424`). It would fail against the pre-fix code: the probe would read true, the commit would skip and HEAD would not move.
3. **Two-repository hub, revert or rebase:** these do not count either. There is no row of their own. They share one code path with the cherry-pick, since the diff removes the `gitOperationAt` call from that branch outright, so row 1 pins them. No finding.
4. **Two-repository hub, a live `MERGE_HEAD` or a conflicted index still counts:** pinned by the existing `TestMergeStateActive` merge steps (`mergestateactive_integration_test.go`: live merge, and the conflicted squash with no `MERGE_HEAD`).
5. **One-repository hub, a cherry-pick still blocks:** pinned by the existing `TestRecordsCommit_DuringMerge/CherryPickRefusesNamingTheOperation` (`recordsduringmerge_integration_test.go:250-265`) and the `TestCommitStatusSeam_Real` row `one-repository cherry-pick in progress waits, then commits` (`wiring_commitstatus_integration_test.go:435`).
6. **A missing path answers false and logs nothing:** pinned by `TestOneRepositoryAt_NoWorktreeOnDiskReadsAsTwoRepositoriesQuietly/MissingPath` (`onerepository_quiet_test.go:26`). It would fail against the pre-fix code, where `dotgit.Read` failed and logged a Warn.
7. **A directory with no `.git` answers false and logs nothing:** pinned by the same test, row `DirectoryWithoutGit` (`onerepository_quiet_test.go:27`).
8. **A worktree's layout is still read correctly through the new `readOneRepositoryLayout` split:** pinned by the existing `TestRecordsGeometry_OneRepository` rows (`onerepository_test.go:141`). The mutation survivor at `onerepository.go:50`, which negates `err != nil`, would flip the `OneRepository` row there. That row is `tier1`, and the mutation run evidently did not reach it, so this survivor is no coverage gap.
9. **A worktree on disk is kept for the process, so a repeated read runs no git** (`onerepository.go:54`, `layoutAnswers.Store`): no test. This is the mutation survivor at `:54`.
10. **A failed read is logged and not kept** (`onerepository.go:49-53`): no test. Before the diff, every failure was already logged, so only the "not kept" half is new.
11. **A fabric marker commit or retirement drops every kept answer** (`forgetLayouts` at `anchormarker.go:85`, `anchormarker.go:211`, `clone_onerepository.go:293`; the body at `onerepository.go:61`): no test. These are the survivors at `anchormarker.go:85` and `onerepository.go:61`. The reach report says no mapped test reaches `seedOneRepositoryBoard`.

**Findings**

- **MEDIUM**, class `design`, at `internal/fabricengine/onerepository.go:54` (`oneRepositoryAt`).
  - **Summary:** No test pins the per-worktree cache that the Goal asks for ("its answer is cached per worktree for the process, so repeated calls run no git"). Dropping `layoutAnswers.Store` leaves every test green: mutation survivor `onerepository.go:54`.
  - **How a test could pin it:** an internal test in `tier1` reads a one-repository worktree (true), then removes the marker with a raw `git rm` and commit outside fabric, and asserts that a second `oneRepositoryAt` still answers true. That observes the kept answer without counting git spawns.
  - **Evidence:** `onerepository.go:49-55`; the mutation report's survivor `:54`; no `_test.go` outside `onerepository_quiet_test.go` and `onerepository_test.go:141` references `oneRepositoryAt`, `layoutAnswers` or `forgetLayouts`.

- **MEDIUM**, class `design`, at `internal/fabricengine/anchormarker.go:85` (`commitAnchorMarker`), also `anchormarker.go:211`, `clone_onerepository.go:293` and `onerepository.go:61`.
  - **Summary:** Invalidating the cache after a marker write is a contract that the `forgetLayouts` doc and `doc.go` state ("committing or retiring a marker through fabric drops every kept answer"), and no test pins it. Dropping the call, or clearing nothing in `forgetLayouts`, passes every test: survivors at `anchormarker.go:85` and `onerepository.go:61`.
  - **How a test could pin it:** an internal test in `tier1` reads a repository with no marker (false, kept), commits the marker through `commitAnchorMarker`, and asserts that the next `oneRepositoryAt` answers true. Without `forgetLayouts`, the stale false stays.
  - **Evidence:** `anchormarker.go:83-86`, `onerepository.go:58-62`, `doc.go` hunk at `+2491-2493`; the mutation report's survivors; the reach report says nothing reaches `seedOneRepositoryBoard`.

- **LOW**, class `design`, at `internal/fabricengine/onerepository.go:49-53` (`oneRepositoryAt`).
  - **Summary:** The doc comment and the summary's design decisions claim that a failed read is not kept, so a transient git failure does not fix a wrong layout for the process. No test pins it: storing on error would pass every test.
  - **How a test could pin it:** a row that reads a path whose `.git` is present but unreadable (for example an empty `.git` file), then repairs it into a one-repository worktree, and asserts that the second read answers true.
  - **Evidence:** `onerepository.go:37-39` and `:49-53`; the "Design decisions" in `_lym/landing/summary.md`.

**Tool calls**
- grep across the `fabricengine`, `loomcli` and `battencli` tests for `cherry-pick`, the probe helpers and any test referencing `oneRepositoryAt`, `forgetLayouts` or `layoutAnswers`: my context did not show which existing tests pin the cherry-pick and the cache.
- grep in `recordsduringmerge_integration_test.go` and `wiring_commitstatus_integration_test.go` for the one-repository cherry-pick rows and the helper definitions: needed to confirm behaviour 5 keeps its coverage.
