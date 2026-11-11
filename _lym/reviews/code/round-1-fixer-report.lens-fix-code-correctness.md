```yaml
lens: fix-code-correctness
outcome: reported
```

Lens fix-code-correctness: run each added test branch step by step, and check that each test fails with its defect in place.

**Findings**

1. **LOW** — `internal/dobbyengine/ended_test.go`, failing-teardown subtest: the comment "the 11th failure reaches the ceiling and the 13th must not pass it" and the message `wait after 13 failures` overstate what is read.
   - The loop exits as soon as `len(fake.calls) == 13`. Call 13 starts at the tick that has just read call 12's result, and its own result is not read yet.
   - So `endedRetries["x"].interval` holds the 12th failure's wait, not the 13th's.
   - The assertion still catches the defect. Without the cap, 12 failures would give 10s·2^11 = 20480s, not 3600s. The 11th failure is the first to hit the cap (10s·2^10 = 10240s), and the 12th shows it stays there.
   - Only the wording is off: say "12 failures read" and "the 12th must not pass it".

**Traced, no defect found**

- **`DobbyRuns` test (`internal/beamcli/dobbypairanchor_test.go`):**
  - It would fail if the `EntryEnded` or `PairPresent` wiring were dropped. `landed` and `unlanded` expect `entryEnded: true`, `pairless` expects `pairPresent: false`, and `ahead` and `unreadable` expect `entryEnded: false`.
  - The tier-1 run passes. Removing the pairless worktree directory does not drop the run from the listing.
- **Removed-pair assertion:** without the `delete(d.endedRetries, …)` loop in `tearDownEnded`, the entry set by the `pair removed` read would survive the `PairPresent=false` ticks, so the test fails.
- **Nil-seam subtest:** with the `d.deps.TearDownEnded == nil` guard removed, the call would panic on the nil func, so it fails. With the guard in place, `teardown`, `inFlight` and `endedRetries` all stay empty.
- **Binary-check subtest:**
  - With `stopAt` at 3, `CheckBinary` is reached after ticks 1 and 2.
  - With the `d.teardown == nil` guard removed, `checks` would be 2, not 0, so the test fails.
  - It passes with the guard in place, which also shows the teardown started in tick 1.
- **Callers:** only test files changed, so no signature or caller is affected.

**Further tool calls**

None.
