```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens. I traced each change in the diff to the Goal points in the decision record. The record has no `Added after Discussion` entries.

**Decisions (Goal points):**
- **G1.** In a two-repository hub the merge-state probe counts only a merge. A cherry-pick, revert or rebase in weft does not make a status commit skip.
- **G2.** The layout probe reads a path with no worktree on disk as two-repository, without a warning. Its answer is cached per worktree for the process, so repeated calls run no git.
- **G3.** Two tests: a status commit during a weft cherry-pick in a two-repository hub commits, and a layout read of a missing path logs nothing.
- **Out of scope:** one-repository behaviour.

**Changes and what they trace to:**
- **C1** — `internal/fabricengine/recordsduringmerge.go`: `MergeStateOperation`'s two-repository branch no longer calls `gitOperationAt`, and its doc comment is rewritten. Traces to G1. The one-repository branch (`recordsMergeKindAt`) is untouched, so it stays out of scope as the Goal asks. The two-repository branch is now `MERGE_HEAD || conflicted`, the same rule as `f06acf39^:mergestateactive.go`.
- **C2** — `internal/fabricengine/mergestateactive.go`: doc comment. Forced by C1; the one-repository sentence restates the existing behaviour.
- **C3** — `internal/fabricengine/doc.go:1351-1353`: `MergeStateActive` paragraph. Forced by C1.
- **C4** — `internal/fabricengine/onerepository.go:45-47`: an `Lstat` of `.git` returns false early, with no git and no log. Traces to G2 (the quiet missing path).
- **C5** — `internal/fabricengine/onerepository.go`: `layoutAnswers`, the cache in `oneRepositoryAt`, `forgetLayouts`, and the `readOneRepositoryLayout` extraction. Traces to G2 (the per-worktree cache).
  - The four Warn sites collapse into one Warn on the returned error. This is forced by C5: an uncached failed read needs the reader to return an error.
  - The cache also covers one-repository worktrees. G2 requires that, because the probe is what decides the layout. It is not one-repository behaviour change.
- **C6** — `forgetLayouts()` calls at `anchormarker.go:85`, `anchormarker.go:211` and `clone_onerepository.go:293`. Forced by C5 (cache invalidation).
- **C7** — `internal/fabricengine/doc.go:2491-2493`: layout paragraph. Forced by C4 and C5.
- **C8** — `internal/loomcli/wiring_commitstatus_integration_test.go`: `startStoppedCherryPickInRecords` and the step `records cherry-pick in progress commits the transition`. Traces to G3 (the status commit during a weft cherry-pick commits).
- **C9** — `internal/fabricengine/mergestateactive_integration_test.go`: the `weft cherry-pick in progress reports false` step. Traces to G1 and G3: it pins the probe change at fabric's own surface. Whether it duplicates C8 is for the tests lens.
- **C10** — `internal/fabricengine/onerepository_quiet_test.go`. Traces to G3 (a missing path logs nothing). The `DirectoryWithoutGit` row is the same G2 case: a path with no worktree on disk.

**Decisions and the changes that build them:**
- G1: C1, with C2, C3 and C9.
- G2: C4 and C5, with C6 and C7.
- G3: C8 and C10, with C9.
- Out of scope: no change alters one-repository merge-probe behaviour.

**Commit messages:** `610cf97d`, `16201d0d` and `5e1c7e4d` describe the traced changes. `7811b3cd` and `e0bec04f` are gate-lint fixes inside the same files. No `Pattern-Departure:` trailer is present, and none is needed.

**Findings:** none. Every change traces to a Goal point or is forced by one, and every Goal point is built.

Tool calls: none.
