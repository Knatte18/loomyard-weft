```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the fix's code changes (the new test table and the comment edits) through the correctness lens.

The fix changed only test code and comments, so there is no changed signature or caller to break. I ran each new row by hand against `recordedAnchorOnWeft` and `checkSubpathBeforeReset`.

- **`MarkerOnlyOnMainWeft`:**
  - The first clone adopts `backend` from `main-weft`.
  - On reset, `fetchWeftBranchTip` reports the branch on the remote, so `primeRef` is `FETCH_HEAD` and the marker is found.
  - `adoptRecordedAnchor` refuses with `does not match the recorded anchor "backend"`, and the sentinel survives.
  - If the `FETCH_HEAD` read were dropped, `main` has no marker and the legacy read finds none. The warp check would then refuse with "does not exist as a directory", so the message assertion fails.
- **`MainWeftWithoutMarkerFallsBackToDefaultBranch`:**
  - The prime read on `FETCH_HEAD` finds no marker.
  - `branch --show-current` returns `main`, which equals `warpBranch`, so the legacy `HEAD` read finds `backend`.
  - If the fallback were removed, the "does not exist as a directory" error would again fail the assertion.
- **`MarkerOnNonWarpDefaultBranchIsNotAdopted`:**
  - `main-weft` is on the remote without a marker, and the default branch is `trunk` with marker `elsewhere`.
  - The prime read finds nothing, and the gate `trunk != main` returns not-found.
  - The warp check accepts `backend`, so the reset succeeds with `res.Anchor == "backend"`.
  - If the gate were removed, the legacy read would adopt `elsewhere` and refuse `backend`, so `t.Fatalf` fires.
  - The tier-1 gate passed all rows (failed 0), so the first clone also works against a `trunk`-default weft.
- **Fixture and isolation:** Each subtest gets its own `t.TempDir()` and the `symbolic-ref` switch comes after all seeding, so `t.Parallel` is safe.
- **Comment edits:** The `t.Parallel` comment above `SecondCloneDerivesShortname` is accurate. All three subtests that follow it use the shared `firstParent` and `first`, and `ResetKeepsRecord` tears that hub down last. The `checkResetTarget` wording in `livestate_doc_test.go` matches `CloneHub` calling it directly.

no findings

Tool calls made: none. Everything this lens needed was already in context.
