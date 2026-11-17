```yaml
lens: fix-code-correctness
outcome: reported
```

Lens fix-code-correctness: I ran each changed function and branch through a concrete input and checked that each added test fails with its defect in place. I made no tool calls.

**Findings**

1. **LOW** — `reset.go`, the `if stopped { … own = append(own, dirty…) }` branch in `Reset`.
   - After `--stop-strand` stops at least one strand, every tracked dirty path is added to `own`, so `refuseForeignPaths` can never refuse.
   - The input is a stopped live attempt plus a dirty `parent-note.go` that the parent edited by hand. `ResetCode` is then handed `parent-note.go` in `own` and discards it.
   - This does what the review's fix asked for, but it turns the foreign-path guard off whenever a strand is stopped. The tests pin only the case where all dirt belongs to the attempt.

2. **LOW** — `run.go`, the `stopErr != nil` branch in `runBatch`.
   - For a NEEDS_PARENT attempt whose strand stop fails, the stuck reason is `stopErr.Error()` and the parent's `Need` text is dropped.
   - The input is a NEEDS_PARENT report with need "run lym fabric merge" and a failing `StopStrand`. The stop is recorded as `needs_parent`, but the reason names only `lym reed remove`.
   - The new test asserts only `lym reed remove`, so it does not catch this.

3. **LOW** — `run_test.go`, `OnlyTheRunsLastBatchIsBuiltAsLastAndTheFirstBatchRefusesARemovedLaterTarget`.
   - The review asked that the first batch's gate spec and prompt carry `LastBatch` false. The row asserts only prompt text.
   - The `--card 1-one --main-packages` and `--card 2-two --main-packages` checks repeat what `ThreeCardPlanRunsTwoBatchesInOrderAndIsDone` already pins.
   - The later-reference gate in the fake depends on `plan.Cards`, `completed` and `own`, not on `IsLastBatch`, so a break in `IsLastBatch` that spares prompt rendering would still pass.

4. **LOW** — `run.go`, `settleAttempt` (not changed by this round).
   - The new D6 test covers a failed outcome with a failed state write: the strand is stopped.
   - A `done` outcome with a failed state write returns before `stopAttempt` (`if out.Outcome == outcomeDone { return out, nil, endErr }`), so that strand is not stopped.
   - Whether that gap matters depends on D6's wording, which I did not read.

**Verified with no finding**
- **F1:** `runBatch` now computes `next, kind` before the `stopErr` check. NEEDS_PARENT gives kind `needs_parent` at the attempt's rung. The new row asserts `Stop.Kind == needs_parent`, which a batch kind would fail. The resume path then takes the needs-parent branch.
- **F6:** `truncateCause` indexes `lead[cut]` only when `len(lead) > 600`, so it cannot go out of range. A 1300-byte cause now keeps the `lym reed remove` tail, and the added `"..."` check holds. The old byte cut would have failed the row.
- **F2 and F7:**
  - `settleRecordedStrands` returns `(bool, error)`; its only caller is `Reset`.
  - The run lock stays held until the deferred release, and `archiveAndClear` keeps `*.lock`.
  - Without the reorder the refusal would carry checkout advice, so the live-attempt row fails. Without the held lock, `activeDuringMove` is false.
- **F8:** the fault case in `nextAttemptRung` is reachable only through `judgeOpenBatch`, because `endAttempt` returns an error for faults in the run path. The new `resume_test.go` row would fail with a climb to `model-b`.
- **F3:** the fake `DetectDrift` uses the report's renames, so dropping them leaves the card unrewritten and the row fails. The batch-2 hash check also covers the re-partition.
- **F5:** the `onGate` hook fires during `Wait`, after the state save that records the strand guid. The later save then fails on `LoadState`, and the strand is stopped once. With no stop, `stops` would be 0.
- **F11:** with the defect, the card hash would stay at its `before` value and the row would fail.
- **F12:** the boolean in the `StateHistory` assertion groups correctly. The test passes at tier 1, per the parent's run.

**Tool calls:** none. The context already held every changed file.
