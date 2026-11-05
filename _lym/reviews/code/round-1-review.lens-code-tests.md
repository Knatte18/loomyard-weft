```yaml
lens: code-tests
outcome: reported
```

Lens: code-tests. Per behaviour, the test that pins it or "none".

1. **Two-argument `--reset` with a disagreeing `--shortname` leaves the hub (the issue's case).**
   - Pinned by `TestCloneHub_ShortnameRecordLifecycle/ResetWithDisagreeingShortnameKeepsHub/TwoArgument` (`shortnamebinding_clone_integration_test.go:94-121`).
   - It fails on the old code, which removed the hub before the probe, because `ReadShortname(first.BoardDir)` would then miss.
2. **One-argument `--reset` with a disagreeing `--shortname` leaves the hub.**
   - Pinned by the `OneArgument` row of the same test.
   - The row also passed before the diff, since the probe already ran first. It guards against a reorder and does not show a fix, which is fine.
3. **Two-argument `--reset` against a non-hub target is refused offline, before the probe, and leaves the contents.**
   - Pinned by `TestCloneHubReset/*/CloneHubReset/NonHubTarget` (`livestate_verbs_test.go:994-1039`).
   - Its weft URL is `https://example.invalid/...`. A non-hub check moved after the probe would fail with "probe weft", not "is not a fabric hub", so the test fails on that regression.
4. **Without `--reset`, the two-argument form fails with "hub already exists" before the probe.** Pinned by `TestCloneHub_HubExistsCheckPrecedesProbeInTwoArgForm` (`warpbinding_clone_integration_test.go:725`).
5. **`--reset` against a real hub still rebuilds it, in both forms.** Pinned by `TestCloneHub_ResetInBothArgumentForms` (`warpbinding_clone_integration_test.go:637`) and `TestCloneHubReset/*/CloneHubReset/RealHub`.
6. **`resetHub` and `checkResetTarget`: refusal of a non-hub, teardown of a real hub, silent no-op on an absent path.** Pinned through `resetHub` by `clone_reset_guard_test.go`, which has the non-hub refusal at line 42, the structural-mark teardown at line 86 and `TestResetHub_AbsentPathIsANoop`. `checkResetTarget`'s `present` return is exercised only through `resetHub`, which is the public path.
7. **A warp-binding mismatch under `--reset` leaves the hub, in both forms.**
   - **Test: none.**
   - `TestCloneHub_ConflictLeavesNoHub` (`warpbinding_clone_integration_test.go:238`) runs without `Reset` and with no existing hub.
   - Nothing runs `Reset: true` with `ErrWarpBindingMismatch`.
   - The decision record names the binding explicitly ("shortname, binding"), so reverting the two-argument reorder for the binding refusal alone would go unnoticed.
   - Finding: MEDIUM, `design`. Evidence: the `Reset:` call sites listed in the grep (`shortnamebinding_clone_integration_test.go:108,124`, `warpbinding_clone_integration_test.go:660,699`, `livestate_verbs_test.go:1025,1064`) never pair `Reset: true` with a mismatching warp URL.
8. **The old-order bootstrap guard under two-argument `--reset` leaves the hub.**
   - **Test: none.**
   - `TestCloneHub_OldOrderInvocationIsRefused` (`warpbinding_clone_integration_test.go:513`) runs without `Reset` against a fresh parent.
   - The summary and the `CloneHub` doc comment both claim the guard precedes the removal.
   - Finding: LOW, `design`. It could be a second row of the finding-7 table.
9. **One-argument form without `--reset` against an existing hub fails with "hub already exists".**
   - **Test: none.**
   - The only test asserting that string is the two-argument one above.
   - The one-argument branch's own stat check was folded into the shared block by the diff, so the diff changed it.
   - Finding: LOW, `design`. Evidence: the grep shows only `warpbinding_clone_integration_test.go:753`.
10. **One-argument `--reset` against a non-hub target is refused, after the probe, with the contents surviving.**
    - **Test: none through `CloneHub`.**
    - `TestCloneHubReset`'s non-hub case is two-argument only.
    - The behaviour is unchanged by the diff: the refusal ran after the probe before it too, and `clone_reset_guard_test.go` covers `resetHub` directly. Not raised as a finding.

Economy: the new subtest adds a row of a new behaviour (item 1) and duplicates no existing test. Two of its three assertions are the shared shortname-refusal text check, which does not repeat `DisagreeingShortnameRefuses`, because that test runs without a hub or `Reset`. No test pins static prose.

Tool calls:
1. Read `livestate_verbs_test.go:985-1075`: my context lacked the `NonHubTarget` fixture's weft URL, to judge whether it pins offline precedence.
2. `grep -rn "hub already exists" --include=*_test.go internal`: to find every test asserting the one-argument or two-argument hub-exists refusal.
