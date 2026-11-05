```yaml
lens: code-tests
outcome: reported
```

**Per-behaviour list (test that pins it, or none)**

1. `PairTaskBranch`, task-worktree step (bare slug read from the worktree, new prefix set): `TestBranchPrefixChange_KeepsLivePairsManaged` (`pairbranch_integration_test.go:36`). It covers the bare slug and the new pair's `hanf/&lt;slug&gt;`.
2. `PairTaskBranch`, records-worktree step: none. Both worktrees stand in the test, so the first step always answers.
3. `PairTaskBranch`, local-branch step and its records-branch half: none. This is the path `Remove` and `Discard` take once the worktrees are gone.
4. `PairTaskBranch`, fall through to `names[0]`: none.
5. `PairTaskBranch`, worktree on a branch outside the pair's names is ignored: none. This is the safety property in the decision record's "Design decisions". No test checks `lym fabric checkout` onto an unrelated branch, and none covers a detached HEAD.
6. `slugOfBranch` and `pairBranchNames`, the collision case (a bare slug beginning with the new prefix): none.
7. `PairParts`: the test asserts it finds the old pair under the new prefix (`:42`). Pinned.
8. `TaskBranchOnOrigin`, bare slug under the new prefix: pinned (`:46`). The test pushes the branch first.
9. `TaskBranchSlugs`: the diff changes the filter from `CutPrefix` with `ok` to `slugOfBranch`. A branch lacking the prefix was formerly dropped when a prefix was set, and is now included. `pairparts_integration_test.go` uses an empty prefix only. No test pins the changed behaviour.
10. `Add`'s parent check in `resolveAddParent` (accepts a bare-slug branch under a prefix): none. No existing test does this.
11. `Remove`, local-remnant path (bare-slug branches deleted under the new prefix, `SiblingBranch` set, the weft bare-slug branch deleted): pinned (`:63-72`).
12. `Remove` / `RemoveRefusal` / `refuseWhenNothingRemains` with nothing local, origin names the pair under either name: none. The test keeps the local branches. This is the focus item in the diff for `inspectPair` and `originPairBranch`. `originPairBranch` and `firstOnOrigin`, including the records-branch-only origin case, have no test.
13. `Remove` makes no origin call when any local part exists: none.
14. Remote teardown job carries `WarpBranch` and `WeftBranch`, with no `BranchPrefix`: `remoteteardown_integration_test.go:99` only drops the `branch_prefix` key from a handwritten job. No test runs a job written under one prefix and executed under another.
15. `DiscardTaskBranch`, `PairTaskBranch` followed by the origin-only fallback (`originTaskBranch`): none. `taskbranchdiscard_integration_test.go` uses an empty prefix, so the new branch never runs.
16. `RemoveResult.SiblingBranch` and the `fabriccli` remove error that reads it (`fabric.go:1119`): the engine value is pinned (`:67`). The CLI error path is untested here. I did not confirm whether `fabriccli` has a test for that message.
17. `ownedManagedBranch` no longer takes a prefix:
    - The `resolveManagedBranch` rejection of a name without `-weft` is covered by the existing `destroy_test.go` rows `:519-538`.
    - `TestBranchOwnership_ManagedBranchKind` accepts a weft-suffixed branch.
    - No row asserts that a prefix-only name is now refused.
    - The ownership narrowing is covered only indirectly.
18. `CleanupRemoteWarp` `keptReason`, dropped "lacks the prefix is unmanaged" refusal: none. `cleanupremotewarp_integration_test.go` uses an empty prefix, so a branch lacking the prefix is never classified.
19. Cleanup orphan (`cleanupOrphan` via `slugOfBranch`) with a prefix set: none.
20. `closePairPullRequest`, gone branch (`PairTaskBranch` read and its failure warnings): `pairteardown/teardown_test.go` names the symbols but, with the changed `taskBranch` argument, I found no assertion that the bare-slug branch is passed after a prefix change. No test pins it. That includes the `gone` branch the focus names.
21. Battencli `hubTaskBranch` (readopt) and `wire.go:1032` (incomplete-pair remedy): none. The only prefix-bearing test is `lifecycle_integration_test.go:1073`, which sets `r4/` from the start with no live bare-slug pair.
22. Signature-only test edits (`destroy_test.go`, `destroyremote_integration_test.go`, `destructivegaps_integration_test.go`, `export_test.go`): they follow the removed parameters and pin nothing new. This is not a finding.

**Findings**

- MEDIUM, `design`: no test covers a prefix set over a live pair where nothing is left locally, so `inspectPair`/`originPairBranch` (`remove.go:206-215`, `:567`) and `DiscardTaskBranch`'s origin fallback (`taskbranchdiscard.go:57-66`) are unpinned. The Acceptance's "teardown" and "remove" are asserted only for the local-remnant path. Evidence: `pairbranch_integration_test.go:20-73` (local branches kept) and `taskbranchdiscard_integration_test.go:2` (empty prefix).
- MEDIUM, `design`: nothing pins that `PairTaskBranch` ignores a worktree on an unrelated branch. That is the safety rule the decision record's "Design decisions" states. Evidence: `pairbranch.go:40`, `:53`, and no test row.
- MEDIUM, `design`: PR lookup after the prefix change, the Acceptance's "PR lookup", has no test. `closePairPullRequest`'s gone branch (`pullrequest.go:39-49`) and battencli re-adoption (`readopt.go:55`, `wire.go:1032`) are unpinned, so the focus question on the gone branch has no test row.
- MEDIUM, `design`: the branch-to-slug steps change behaviour with a prefix set and have no prefixed test: `TaskBranchSlugs` (`pairparts.go:74`), `keptReason` (`cleanupremotewarp.go:197`), `cleanupOrphan` (`cleanup.go:273`) and `resolveAddParent` (`addparent.go:48-65`). The Acceptance names listing and cleanup.
- LOW, `design`: no test pins `PairTaskBranch`'s records-worktree step, its local-branch fall-through, or `names[0]` (`pairbranch.go:44-66`). A regression there would pass.
- LOW, `design`: no test pins the remote teardown job carrying its own branch names under a different prefix (`remoteteardown.go:62`, `taskbranchdiscard.go:79`). The only edit is the removed JSON key (`remoteteardown_integration_test.go:99`).

**Test economy:** `pairbranch_integration_test.go` is a single test and repeats no existing test I found, so I raise no redundancy finding.

**Tool calls (1):** `grep` over `*_test.go` for the changed symbols and prefix-setting rows, because the manifest held only the diff's test edits and no existing tests.
