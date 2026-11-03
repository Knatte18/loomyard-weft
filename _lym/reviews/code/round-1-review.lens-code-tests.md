```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, I name the test that pins it, or none.

**Per behaviour**
1. `bootstrapCommitPaths(loc, withFriction)` adds the friction directory to the pathspec only when set. Pinned by `TestBootstrapCommitPaths`, rows `NoFrictionDirectory` and `FrictionDirectoryPresent` (`internal/loomcli/sharedbootstrap_test.go:185-212`). It replaces the old single-case test, so nothing is duplicated.
2. The first-seed clear of a tracked inherited note is committed with the seed, so Preflight passes. Pinned by the `first seed clears friction notes` subtest of `TestSmokeStepBootstrapWiring`, which now commits the stale note first and asserts the persisted status is neither blocked nor failed (`internal/loomcli/smoke_bootstrapwiring_test.go:61-101`). It would fail if the deletion were left uncommitted.
3. `AnchoredPathHasContent`, true case (tracked file deleted from disk), has only indirect coverage. The same subtest exercises it through `seedAndCommitBootstrap`. No test names it, and none in `fabricengine` covers it.
4. `AnchoredPathHasContent`, false case (empty or absent friction directory, which keeps git from refusing a matchless pathspec). I found no test. The `first seed` subtest has a deleted tracked note, so its probe is true. The `re-entry` subtest plants an untracked note first, so its probe is also true. Nothing drives the probe false through `step`. If the `withFriction` guard were dropped, no row here would fail, except possibly other bootstrap tests I did not read.
5. `AnchoredPathHasContent`, ignored-only match: no test (`internal/fabricengine/commitweftpaths.go:107`). It relies on `entryMatchesWeft` passing `--exclude-standard`. `weftgit_exclude_test.go` covers that for `weftPathspecFilter`, not for this new entry point.
6. The probe error path returns `bootstrapStageCommit` (`sharedbootstrap.go:166-169`). No test. This is internal git failure, so I rate it low.
7. Probing on every invocation commits untracked friction notes on a resume (`ErrSeedExists`), which is new behaviour. The `re-entry keeps friction notes` subtest checks only that the notes remain on disk (`smoke_bootstrapwiring_test.go:136-`). It does not check whether they are committed, so the choice to commit resume notes is unpinned.
8. Self-heal after a crash between the clear and its commit: no test.
9. The `docs/overview.md` and `doc.go` additions are prose and exempt.

**Findings**

- **LOW, scope.** `internal/fabricengine/commitweftpaths.go:107` (`AnchoredPathHasContent`), plus the guard at `internal/loomcli/sharedbootstrap.go:164-171`. The false branch (empty or untracked-empty directory, or ignored-only content) has no pinning test. Fix: add a table-driven test over a real records worktree with rows for tracked-deleted, untracked, ignored-only and empty directory, or add a `step` smoke subtest with an empty friction directory.
- **LOW, scope.** `internal/loomcli/smoke_bootstrapwiring_test.go:136`. The re-entry subtest does not assert whether the resume's untracked note is committed, so the changed resume behaviour (point 7) is unpinned. Fix: assert the commit state of `resumePath` after the second `step`, whichever outcome the design intends.

**Tool calls:** none. The manifest already held the diff, `entryMatchesWeft` and its callers, and the smoke test.
