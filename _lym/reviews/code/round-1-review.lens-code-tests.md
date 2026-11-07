```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens. For each behaviour the diff changes, the test that pins it, or none.

**Per behaviour**

1. **Add's launcher blocker is a directory.** Pinned by `assertArrangePreconditions` in `runCell` (`livestate_matrix_test.go:~195-215`), but weakened. The check runs only when `fixture.LauncherBlocker != ""`. The deleted test called `os.Lstat("")` unconditionally, so an Arrange that stopped setting the field failed it. Now the Add column goes vacuous silently. No row would fail.
2. **Checkout's branch differs from the original.** Pinned in `runCell` and in the Effect at `livestate_verbs_test.go:763`. The `CheckoutBranch != ""` guard has the same weakening. The Effect backstops it for the clean state only.
3. **Pull's arrange advanced the warp bare.** Pinned in `runCell` and in the Effect at `livestate_verbs_test.go:947`. The `AdvancedToSHA != ""` guard has the same weakening. The old test failed on `"" == ""`, and the new one skips it. The Effect backstops the clean state only.
4. **Preconditions run right after arrange and before the state is applied.** Pinned by call order in `runCell` only. No test would fail if the call moved after `state.Apply`.
5. **`TestAnchorChange`'s three setup-free refusals share a hub.** Pinned by the shared subtest rows through `requireAnchorRefusal`, which asserts the refusal text, weft HEAD and status, wiring, and move record. Every row left in the planted table has a `setup` (the grep lists a `setup:` for each name), so the removed `tt.setup != nil` guard is safe.
6. **`TestStoreHousekeeping`: the default (nil) probe packs the stores.** None. This is the "Remove packs a store above the lowered threshold" row, which the diff deleted. The report's alone-pins column names it ("that the default probe packs"). The surviving rows either install a probe or lower no threshold:
   - The gc-prune row now installs an explicit empty probe, so it covers the probe-answered path, not the nil-probe path.
   - "Remove writes both keys into both stores" asserts only the keys and has no threshold or packing assertion.
   - The two skip rows assert `wantPack` false.
   - `housekeeping_integration_test.go:191` is the only other packing assertion, and it uses the empty probe.
   - The summary says the nil-probe default "still runs" under the key-write row, but that row asserts nothing about packing.
7. **Empty probe answer packs the stores and writes both keys.** Pinned by the merged gc-prune row (`housekeeping_integration_test.go:~191-215`: `requireHousekeepingKeys` for both stores, `sampledLooseObjects == 0`, old object pruned, young object kept).
8. **`owned_elsewhere` runs six replies on one pair.** Pinned by the existing per-subject assertions, which still run each iteration. `asked` is reset per iteration. The commit says no origin moves.
9. **Rewound hold-back on the settle rows' first push.** Pinned by `requireRewoundHoldBack` in `SettledByResettingToOriginsHead` and `SettledByAPlainPushThatRestoresTheDroppedCommits`. It asserts the error class, record head and commits, way forward, origin tip, one record and one mail.
10. **Transport backoff table.** Pinned by two rows, `ADetachedPrimeWeftPush` (nil error) and `AParkedPairPush` (transport `*PushFailedError`). Each asserts one transport record, the full backoff, and one mail. The mail assertion is new on the pair row.
11. **Behind-only pulls.** Pinned by two rows. The one-write row asserts the record kinds and the result-record length. The two-write row asserts the subjects `write b` over `write a`. The common checks also run on both rows.
12. **Seed drops.** Pinned by two rows. The rewritten-seed row asserts one landed commit, one commit past upstream, and `commits_dropped == [oldSeed]`. The out-of-subtree row asserts the commit is replayed under its message and its content.
13. **Diverged conflicts.**
    - Both rows assert the diverged skip, the unchanged head, a clean board and an empty record.
    - Only the non-seed row asserts the way-forward text, that it names no `pull --rebase`, and that the detail is logged once.
    - The logging row is kept, but `logs` is one capture shared by two parallel rows. Counting `res.SkipDetail` is safe because the seed row skips its count and the details name different commits.
14. **The `TestVerbCases_CleanState` clean-state error shape, Effect and manifest diff.** Pinned by `TestCrossProduct` clean cells. The cell-tally assertion at `livestate_matrix_test.go:~303` covers the omissions.

**Findings**

- **MEDIUM, design.** `internal/fabricengine/housekeeping_integration_test.go:~145-215`. No row pins that a removal with no probe set (the default) packs the stores above the lowered threshold. The deleted "Remove packs a store above the lowered threshold" row alone pinned it. The commit message for the housekeeping fold lists only the empty-answer assertions as moved, and the summary's claim that the nil-probe path "still runs" is not an assertion of packing. Evidence: the `SetGCAutoThresholdForTest` calls at `:166` and `:191` both use a probe; the grep shows no other test calls it. Fix: have the gc-prune row leave the probe unset, which is what it did before the fold, and keep the empty-probe packing assertion on a skip-table row (`wantPack: true`), or add the nil-probe `sampledLooseObjects == 0` check to that row.
- **LOW, design.** `livestate_matrix_test.go:~195-215`. The three guards (`LauncherBlocker`, `CheckoutBranch` and `AdvancedToSHA` against `""`) let a verb whose Arrange stops setting its field pass silently, which the deleted per-verb switch would have failed. Fix: make the check fail when the field is empty for its verb. That needs the verb name or a per-case flag, or a `Verbs`-table test asserting that Add, Checkout and Pull each return the field from Arrange.

**Tool calls (1)**
- One Bash: grep for the threshold-setter and loose-object helpers across the package tests, and the `name:`/`setup:` lines of `TestAnchorChange`'s planted table. My context had no view of other packing tests or of the planted rows' `setup` fields.
