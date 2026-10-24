```yaml
lens: code-correctness
outcome: reported
```

Task: lens code-correctness, running each changed function and branch on concrete inputs.

**List (case run → result)**

- `RecoveryRetry`, nil state or missing batch → `false, 0`. Correct, and unchanged.
- `RecoveryRetry`, fork-kind record that is terminal and dead with `Recoveries` 0 → the kind check returns `false, 0`. Correct: the new early return sits after the kind, terminal, dead and cap checks.
- `RecoveryRetry`, non-terminal recovery record, or status stuck or failed, at 0 → `false, 0`. Correct.
- `RecoveryRetry`, terminal dead recovery, `Recoveries` 0, empty `RecoveryStartSHA` (the reset state) → `true, 0`. Correct, and it makes no git call. The empty-start false is unreachable at 0 but is still reached at 1.
- `RecoveryRetry`, `Recoveries` 0 with a non-empty start (a spawn that counted nothing because it rendered an amendment) → `true, 0`. This is correct per the decision record and bounded. The retry spawn re-renders the same amendment, which is no longer new, so it counts and `Recoveries` becomes 1. From there the commit rule applies.
- `RecoveryRetry`, `Recoveries` 1 with an empty start → `false, 1`. Correct, and unchanged.
- `RecoveryRetry`, `Recoveries` 1 with a start → `committedSince`, including its failure path (`false` plus a Warn). Correct, and unchanged.
- `RecoveryRetry`, `Recoveries` 2 or more → `false`. Correct. The cap is still checked before the zero branch, so a retry can never bypass it.
- `existingReportRemedy`, dead record with `retry` true → names `recover-batch N`, returns `exhausted` false. Correct. The exhausted branch runs first and needs `Recoveries` >= 2, so it cannot collide with the zero case.
- `existingReportRemedy`, dead record with `retry` false (`Recoveries` 1, no commits) → still "earns no retry". Correct.
- `begin-batch` caller (`beginbatch.go:556`) → passes the verdict through unchanged. Correct. A re-begin carries `Recoveries` and the start forward (`beginbatch.go:641-656`), so no new loop path appears.
- `status.go` and `webstercli/recoverbatch.go:344` → both read `RecoveryRetry` alone. Correct. A dead record at 0 now gives `recovery_retry: true` in both, and the error-envelope message says "once more", which matches.
- Stencil dead rung → the dropped "that second recovery is the last" is correct now that a reset can grant a fresh allowance. The code cap bounds the loop.

No findings. I found no wrong result, no misreported error and no inconsistent state on a failure path.

**Tool calls:** none. The manifest already held every definition, caller and the reset state writes I needed.
