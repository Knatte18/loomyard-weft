```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the correctness of the code the fix changed (fix-code-correctness).

No `BLOCKING` or `MEDIUM` findings.

1. `LOW`: `internal/landingshed/finalize.go`, `gateUnlocked` (`!fz.needsGitHubOnDisk()`). The F4 premise still holds for a landing that needs GitHub only because the repository's rules require a pull request.
   - `needsGitHubOnDisk` passes `false` for the rules-required input to `ResolvePullRequest`, and answers `false` for modes other than `trail` and `review` when no pull request is recorded.
   - For such a first landing, with an owed parent, the lock-free pass still merges and gates the owed tips. `parentPushBlocks` under the lock then ends the call.
   - Evidence: the `needsGitHubOnDisk` body, and `doc.go`: "GitHub's rules that require a pull request count as required whatever beam.yaml says".
   - The new comments say "known to need GitHub", so they stay accurate. This is a residual limit to record or accept.

2. `LOW`: the same line, against the `needsGitHubOnDisk` doc ("A failed read answers false, since the decision under the lock makes the same read and ends the call on it").
   - If `pr.Mode`, `pr.Origin` or `pr.TaskHasCommits` fails on a GitHub landing, the answer is `false`.
   - The owed tips are then merged into the task branch before the locked decision ends the call, so the merge stays after the error.
   - The sentence's justification covered only the old use, which was skipping an early push check.

Traced with no defect:
- **`parentMergeTip`, owed true then true:** it returns the local tip.
- **Owed true then false:** it returns the upstream tip read before the lock.
- **Second read fails:** it returns the wrapped error, and the deferred release frees the lock.
- **Local tips read fails or the lock wait pauses:** behaviour is unchanged.
- **Push skipped or no upstream:** `owedUpstream` stays empty, so there is no second read.
- **Owed push completes between the two reads:** a level branch reads `true`, so the local tip is returned.
- **Nil seam:** it is never called through the closure.
- **Parent-Align caller:** it passes `false` and is unaffected.

Tests:
- The "commit arrives before the lock" row and the F3 row were both seen failing with the defect in place.
- The "ahead by more than owed" and "unwired" rows fail if an ahead parent returns `tips.Local`.
- Removing only the first `!owed` early return would still pass, because the second read yields the same tip. The merged tip is the same either way.
- F4 has no test, so reverting `!fz.needsGitHubOnDisk()` to `true` would pass the suite. The fix added no test there, so this is a note and not a finding under this lens.

Tool calls: none.
