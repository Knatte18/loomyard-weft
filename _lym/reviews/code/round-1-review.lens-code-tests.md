```yaml
lens: code-tests
outcome: reported
```

Lens: code-tests. For each behaviour the diff adds or changes, the test that pins it, or none.

**Per behaviour**
1. `Remove` returns an error wrapping `ErrStrayPathKept` that names the directory. Pinned by `TestRemove_FailsOnTaskWorktreeDirRefilledAfterItsRemovalThatCannotBeDeleted` (remove_leftoverdir_integration_test.go:222-228).
2. `Remove` finishes the teardown and then returns the error. Pinned only partly: that test keeps its later assertions on the directory and `StrayPath`. No row checks that the branch steps and housekeeping still ran before the error.
3. `Remove` skips `respawnPairPush` for `ErrStrayPathKept` (remove.go:230). Pinned by none. Nothing in the leftover-directory tests exercises respawn.
4. `strayErr` returned from the `!out.ran` return, and `out.err` taking precedence over `strayErr` (remove.go:~400-424). Pinned by none. The kept-directory test passes `Remove(l, slug, false, false)`. Whether that run reaches the handed-over remote path is unclear, and the `out.err` precedence has no row either way. The summary admits the `localMode` return has no test.
5. `PairParts.StrayPath` is set from `inspectPair`, and `Any()` counts it (pairparts.go). Pinned by none. The `PairParts` tests in the cancel code use a fake parts value, so a broken `inspectPair` mapping or `Any()` would pass every test.
6. `RemoveStrayTaskPath`. Pinned by `TestRemoveStrayTaskPath`, which covers delete, absent path, registered worktree refused, `.git` refused and failed delete wrapping `ErrStrayPathKept`. Slug validation and the prime-slug refusal have no row. That is a minor gap, because they are the guard against deleting a path outside the task worktree.
7. `pairteardown.clearStray`. Pinned by `TestRemovePair_ClearStray`. It covers:
   - with and without the flag;
   - success;
   - a kept directory the delete clears, including `StrayPath` reset to "";
   - `ErrPairNotFound`;
   - a failed delete;
   - a refused removal;
   - the delete running before the claim is settled.
   
   Only the production wiring of `removeStray` through `New` is untested, which is trivial.
8. Worktree-Teardown's `teardownRequest` sets `ClearStray` (wire.go:408). Pinned by none. The grep finds `ClearStray` only in cancel_test.go, so dropping it from `teardownRequest` would fail no test, and the resumed-teardown path rests on it.
9. `teardownRefusal` rewords `ErrStrayPathKept` with the resume message. Pinned by wire_test.go (the new block in `TestTeardownRefusal_RecordsRemedy...`).
10. `cancel --remove` sets `ClearStray` (pinned, cancel_test.go:342 and 700). Cancel without `--remove` refuses on a stray-only directory and names `--remove` (pinned by `StrayDirectoryAloneRefusesWithoutRemove`). Cancel with `--remove` proceeds on a stray-only directory (pinned by `StrayDirectoryAloneWithRemoveDeletesIt`). The old warning is removed, and the replaced test shows nothing is lost. A failed delete through `removePairAndBranch` reuses the generic refusal path, which is covered by the existing refused-removal tests.
11. Beam-land `HasPair` reads `PairWorktreeStands` (beamland.go:127). Pinned by none that I can confirm. beamland_test.go references `beamMembers`/`BeamLandMember`, but the diff touches no beamland test, so no row feeds it a leftover directory.

**Findings**
- **MEDIUM, design:** `teardownRequest` has no test that `ClearStray` is set. Batten's Worktree-Teardown depends on it to clear the directory on a resume. Location: internal/battencli/wire.go:408. Evidence: grep of `internal/battencli/*_test.go` finds `ClearStray` only in cancel_test.go.
- **MEDIUM, design:** `PairParts.StrayPath` and `Any()` are untested against a real `inspectPair`. Location: internal/fabricengine/pairparts.go:25-46. Evidence: the only `StrayPath` assertions outside the leftover-directory test are `res.StrayPath` in remove_finish_integration_test.go:229, which reads `RemoveResult`, not `PairParts`. Cancel's stray-only branch depends on this mapping.
- **MEDIUM, design:** the beam-land member listing change has no row with a non-worktree directory at a member's path. Location: internal/battencli/beamland.go:127. Evidence: no beamland test appears in the diff.
- **LOW, design:** the `respawnPairPush` skip for `ErrStrayPathKept`, and the `strayErr` versus `out.err` precedence in the remote and non-`localMode` returns, have no row. Location: internal/fabricengine/remove.go:230 and :400-424. Evidence: the integration test calls `Remove(..., false, false)` and asserts nothing about respawn or `out.err`.
- **LOW, design:** `RemoveStrayTaskPath`'s slug and prime-slug refusals have no row in `TestRemoveStrayTaskPath`. Location: internal/fabricengine/destroy.go:1420-1426.

**Tool calls**
1. grep for `teardownRequest`/`ClearStray` in the battencli tests, `beamMembers`/`BeamLandMember` test files, `StrayPath` in the fabricengine tests, and respawn/`ErrStrayPathKept` in the leftover test: the manifest held only the diffs of the changed tests, not the unchanged tests.
