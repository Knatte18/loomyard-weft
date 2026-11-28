```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens. For each behaviour the diff adds or changes, I looked for the test row that fails when that behaviour breaks.

**Behaviour → pinning test**

1. **Finalize's lock-free pass, parent ahead by owed tips alone: merges the local tip.** Pinned by the `TestFinalize_GatesWithNoLockAndLocksToLand` row "a parent ahead by owed landing tips lands on its local head gated with no lock" (`internal/landingshed/finalize_unlocked_test.go:75`). Return the upstream tip instead and `wantTips` `["p1",""]` fails, as does `wantLockAtVerify` `[false]`.
2. **The local tip is read under the parent lock.** This is shared code after the switch (`parentalign.go:265-274`). The `lockHeld: true` assertions of the `TestMergeIn_MergeTip` rows "the push skip holds the lock…" and "no upstream holds the lock…" pin it (`parentalign_test.go:344-352`). The owed row does not assert it itself, but the code path is the same.
3. **Finalize's lock-free pass, parent ahead but not owed-only (`ParentAheadByOwed` answers false): merges the upstream tip.** No test.
4. **Finalize's lock-free pass, parent ahead with a nil `ParentAheadByOwed`: merges the upstream tip.** No test.
5. **A failed `ParentAheadByOwed` read in the lock-free pass: returns an error wrapping the cause, and nothing is merged.** Pinned by the `TestFinalize_LockFreePassStops` row "a failed read of whether the parent is ahead by owed landings alone" (`finalize_unlocked_test.go:280-289`). It asserts the wording, the wrapped error, no merge-in (`res.tips` empty) and no parent-side merge (`finalize_unlocked_test.go:318-325`).
6. **Parent-Align passes `takeOwedLocal=false` and keeps the upstream tip for an owed-ahead parent.** No test.
7. **Unchanged relations (equal, behind, diverged) still take the upstream tip.** The existing rows still pin them: `TestMergeIn_MergeTip`'s behind and diverged rows (`parentalign_test.go:328-342`) and `finalize_unlocked_test.go:69,74`.

**Findings**

- **MEDIUM, design.** `internal/landingshed/parentalign.go:247-257`: Finalize's handling of an ahead parent that is not owed-only, and of a nil seam, has no test that fails when it breaks.
  - If the `ParentAhead` case returned `tips.Local` unconditionally, both the upstream fallback for `owed == false` and the nil-seam fallback (`!takeOwedLocal || deps.ParentAheadByOwed == nil`) would break, and every test would still pass.
  - Evidence: the only `ParentAhead` rows that reach the lock-free pass are the owed-true row and the error row (`finalize_unlocked_test.go:91-96,283-286`). `finalize_test.go:1150` drives `syncParent` alone, not `parentMergeTip`.
  - Fix: add a `TestFinalize_GatesWithNoLockAndLocksToLand` or `TestMergeIn_MergeTip` row where the parent is ahead and `ParentAheadByOwed` answers false, asserting the upstream tip is merged. Add a row with the seam nil as well.
- **MEDIUM, design.** `internal/landingshed/parentalign.go:147`: no test pins that Parent-Align ignores an owed-ahead parent and merges the upstream tip.
  - `newMergeInFixture` (`parentalign_test.go:101`) never sets `ParentAheadByOwed`, so the seam is nil. Changing the call to pass `true` would still hit the nil fallback, and the ahead row "a local parent ahead of its upstream merges the upstream tip, never the local-only commit" (`parentalign_test.go:334-337`) would still pass.
  - Evidence: grep finds `ParentAheadByOwed` set only in `finalize_unlocked_test.go:96,286`.
  - Fix: in that ahead row, or a new one, set `fx.m.deps.ParentAheadByOwed` to answer true and assert that `wantTip` stays the upstream `p1`.

**Tool calls**

1. I grepped for every test that sets `ParentAheadByOwed`, for the location of `newMergeInFixture`, and for the assertion body of `TestFinalize_LockFreePassStops`. The manifest had not shown the fixture's seam wiring or that test's assertions.
