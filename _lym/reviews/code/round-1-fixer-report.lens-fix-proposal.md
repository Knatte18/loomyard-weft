```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals for F1–F3 as the fixer carried them out.

Proposals checked:
- **F1:** the `DobbyRuns` rows hold up. Each wiring has a row that fails if it is dropped.
  - `EntryEnded` is pinned from both sides: `landed` is done, `unlanded` and `pairless` are gone, and `ahead` and `unreadable` stay open.
  - `PairPresent` is pinned by `pairless`, whose pair directory was removed.
- **F2:** both rows hold up.
  - The binary-check row would see 2 `CheckBinary` calls (after ticks 1 and 2) if the `d.teardown == nil` guard were dropped, so it fails.
  - The nil-seam row would panic on a nil `TearDownEnded` call if its guard were dropped, so it fails too.
- **F3:** both assertions hold up.
  - Without the `min(..., endedTeardownCeiling)` cap, the wait after 13 failures would be 10s·2¹² = 40960s, not 3600s, so the ceiling assertion fails.
  - Without the `delete` of a no-longer-stranded wait, `endedRetries["x"]` stays set, so the removed-pair assertion fails.

Findings:

1. **MEDIUM** — comment in the failing-teardown subtest, `internal/dobbyengine/ended_test.go`. It reads `// The wait doubles from 10s, so the 11th failure reaches the ceiling and the 13th must not pass it.`
   - **Problem:** the comment gives the wrong failure count. The wait after failure n is 10s·2^(n-1). Failure 9 gives 2560s, and failure 10 gives 5120s, which is capped to 3600s. So the ceiling is reached on the 10th failure, not the 11th.
   - **Evidence:** `readEndedTeardown` in `ended.go` sets `interval = probe` on the first failure and `min(retry.interval*2, endedTeardownCeiling)` on each later one.
   - **Effect:** the test's behavior is unaffected, since 13 failures are still enough to catch a missing cap. The comment states a count the code does not produce.

2. **LOW** — the ceiling assertion, same subtest. It checks only `endedRetries["x"].interval`, not that `next` is `now + interval`.
   - **Problem:** a regression that stored the right interval but scheduled `next` with a different value would pass.
   - **Evidence:** the loop advances to `endedRetries["x"].next` each time, and the final check reads only `.interval`.
   - **Effect:** a minor gap, not a wrong result.

Tool calls: none; my context already held every changed file.
