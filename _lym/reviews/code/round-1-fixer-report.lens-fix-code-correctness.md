```yaml
lens: fix-code-correctness
outcome: reported
```

No findings.

Task: run the correctness lens over the F1–F5 change set.

I ran each changed path with one concrete input and found nothing wrong.

- **`ClearDoneRun` and `discardTaskBranchIfPresent`:** The cancel path also passes the prime location to `DiscardTaskBranch`, so the call is wired the same way. A done run with a leftover branch discards the branch, then removes the records. A discard failure returns before any record is touched, so the next tick retries. If the records commit fails after the branch is gone, the re-run finds no branch (`PairParts` and `TaskBranchOnOrigin` both false) and goes straight to the records. A failed `hubTopology` returns an error before any lock is taken.
- **`clearDoneRun` callers:** The motif path passes `nil` and the two tests were updated. `ClearDoneRun` passes the discard function. No caller is left on the old signature.
- **`clearDoneRuns` stale drop:** `cleared` slugs are listed, so they are never added twice. A slug that is landed or listed is kept. `d.landed` is set by `countLandings` earlier in the same tick. A stale entry such as `gone` is dropped even when the earlier update failed, and the `dropped := cleared` aliasing is harmless.
- **Tests fail with the defect in place:**
  - `TestClearDoneRuns_WaitsForTheLandingCountOfARunWhileATidyIsInFlight`: without `d.landed = landed` in `countLandings`, the clear is called during the in-flight tidy, so the first assertion fails.
  - `TestClearDoneRuns`: the `gone` and `other` assertions fail if the stale-drop loop or the listed check is removed.
  - `TestClearDoneRun_RemovesADoneRunTheRecipeFilterAdmits`: `wantDiscards` fails if the discard call is dropped or moved after the removal. The new `discardErr` row fails if its error is swallowed or the records are removed anyway. `wantMessage` pins both commit-message branches.
- **Gap, not a defect:** the `d.clearDoneRuns(now)` call in `tick` is still unpinned. Only `countLandings` followed by `clearDoneRuns` is driven, which is what F3's fix key allows.

Further tool calls (1): `sed` on `internal/battencli/cancelcmd.go:165-190` plus a grep of `ResolveBattenPrime`, to confirm the cancel path also passes the prime location to `DiscardTaskBranch`; my context lacked how `hubCancelDeps` gets its location.
