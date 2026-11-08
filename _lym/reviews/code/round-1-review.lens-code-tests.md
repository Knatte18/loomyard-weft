```yaml
lens: code-tests
outcome: reported
```

**Task:** code-tests lens. For each behaviour the diff adds or changes, the test that pins it, or none.

**Per behaviour**
- `parseTrackedDirtyPaths`: pinned by `TestParseTrackedDirtyPaths` (`dirtiness_parse_test.go`). It covers empty output, a plain path, spaces and non-ASCII, and the rename and copy skip. `TestSourceDirtyPaths` (`sourcedirty_integration_test.go:24`) stays as the git-backed test, so nothing was removed there.
- `entriesFromWorktrees` and `parseWorktreePorcelain`: pinned by `TestEntriesFromWorktrees` and `TestParseWorktreePorcelain`, which cover Main on the first entry, the branch-prefix strip, detached, prunable and the bare-repo error.
- The real-git `List` path: only partly pinned. `TestList` (single and two-worktree rows) is removed. `worktreelist_test.go` now calls `fabricengine.List` at lines 33, 63, 123 and 190, and `add_hubreader_integration_test.go:31` calls it too. See F1.
- `excludeLinesToStrip` and `stripExcludeLines`: pinned by `TestExcludeLinesToStrip`. Its five rows cover the three removed git rows plus no sibling and another name's line.
- The wiring of those two helpers into `unseedGitExclude` (the `l.AnchorRel` argument and the `names` input): see F2.
- `assertArrangePreconditions(verb)`: pinned by the cross product, which calls it for every cell. The empty-field `Fatalf` fires only if a verb's Arrange regresses, and no test feeds it an empty field. The record asks only that it be made to fail on an empty field, which it is. The `switch` skips silently if a verb's `Name` is not exactly `Add`, `Checkout` or `Pull`. The removed test looked verbs up by those same names, so this is not a regression.
- `withVacuityCheck`: pinned by the cross product, since every structural-state Apply goes through it. Its first-Apply-per-anchor skip is a deliberate narrowing, in the commit message and the doc comment. A later no-op Apply at the same anchor goes unchecked. No finding.
- The removed `TestStates` per-state establishment assertions: moved inline, as the `tb.Fatalf` lines in `livestate_states_test.go` show.
  - The dirtiness states check the sentinel and the shape prefix (lines 178 and 186).
  - The two link states check the link, with the stale target gone (`livestate_states_test.go:375-381`).
  - The foreign directory checks the planted content (line 405).
  - The unrelated clone checks it is clean (line 431).
  - Three assertions I found no replacement for: `foreignDir` is not a git checkout, the unrelated clone has a commit, and `trackedSymlink`'s raw target exists and is git-tracked. See F3.
- Merged `TestCleanup_VerdictIsIndependentOfDryRunAndForce`: all assertions are kept. These are dry-run parity, orphan deletion, and the force=true apply deleting the orphan while the unmanaged branch survives. No finding.
- `TestWireJunctions_ScratchLink`: the nested-anchor rows keep every assertion of the dropped root rows. The root anchor is covered by the remaining non-looped rows, which are not shown in the diff.
- `TestCommitWeft_MachineLocalArtifactsNeverEnterWeftTreeAtNestedAnchor`: the `wts/some-task` row keeps the durable-state and forbidden-name assertions. The `.` and `backend` rows are dropped, as the record asks.

**Findings**
- F1, LOW, design, `worktreelist_test.go` and `worktreelist_parse_test.go`: the removed `TestList` was the only check of two things. `List` returns the main entry's path equal to the hub path after `filepath.FromSlash` normalisation (the removed lines checked it), and `List` returns a non-empty `Head` from real git. The tier 0 tests use synthetic paths and heads. I did not open `worktreelist_test.go:63` or `:123` to see whether either call asserts these.
  Fix: add the path and head assertions to the surviving real-git `List` test, or confirm that call already makes them.
- F2, LOW, design, `unwire_test.go` (`TestUnwire_LeavesSiblingWorktreeUndirtied`) and `unwire_exclude_test.go`: the `wantKept` and `wantStripped` assertions on the real exclude file are removed. The tier 0 test pins the helper logic only. The surviving git test checks sibling `git status` equality, which catches over-stripping but not under-stripping.
  - Under-stripping of the unwired pair's own line would go unnoticed here. It is covered only if `junction_pattern_integration_test.go:203` runs at a nested anchor. I saw it assert the bare `_lym` line gone, not the anchored `/sub/_lym` line.
  - A wrong `l.AnchorRel` argument in `unseedGitExclude` would also go unseen.
  Fix: keep one `wantStripped` assertion, for example `/sub/_lym` in row `UnwiredPairAloneAtItsAnchor`, in the git test.
- F3, LOW, consistency, `livestate_states_selftest_test.go` and `livestate_states_test.go`: the removed `TestStates` checked that `foreignDirAtFabricOwnedPath` is not a git checkout, that `unrelatedGitCloneAtWeftNamedPath` has a commit, and that `trackedSymlinkAtWiredPath`'s raw target exists and is git-tracked. I found no inline `Fatalf` for these in the grep of `livestate_states_test.go`; I did not read lines 309-335 or 415-431 in full. If a state builder drops one, the cross-product cells could go vacuous without a failure.
  Fix: confirm the three are asserted inline. If not, add them to those states' Apply, which is also where the record puts the vacuity check.

**Tool calls**
1. `grep` for `fabricengine.List(`, exclude-file assertions in the unwire and junction tests, and `Fatalf` lines in `livestate_states_test.go`: my context lacked the surviving `List` callers and the inline establishment checks.
2. `sed` of `reconcile_stale_registration_test.go:596-640`, plus a `grep` for `TestSourceDirtyPaths`: the merged cleanup test's tail was cut off in the diff, and I needed to confirm the dirty-path git test was untouched.
