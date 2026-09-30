# Card 1 — shedengine-budget-exempt

**Edit:**
- `internal/shedengine#OutputPointer`
- `internal/shedengine#HistoryEntry`
- `internal/shedengine#Shed.stepLocked`
- `internal/shedengine#episodeStuckCount`

**Create:**
- `internal/shedengine/budgetexempt_test.go`

**Intent:**
Let a producer mark a Stuck verdict as not counted against its row's bounce budget, so a wait with its own bound (an awaiting child, a driver-exit grace) never exhausts `max_bounces`.

- `OutputPointer` gains `BudgetExempt bool`, documented as read only on a Stuck verdict and ignored on every other outcome.
- `HistoryEntry` gains `BudgetExempt bool` with the JSON tag `budget_exempt,omitempty`, so a status file written before this change decodes as not exempt and counts exactly as today.
- The `appendHistory` closure inside `Shed.stepLocked` copies `output.BudgetExempt` onto the appended entry when the outcome is Stuck, and leaves it false otherwise.
- `episodeStuckCount` skips an entry whose `BudgetExempt` is true; its doc comment says so.
  Routing is unchanged: an exempt Stuck still follows `OnStuck`, and a row with no `OnStuck` still blocks on it.

The field is plain data on both structs, so the Shed Producer-Seam Invariant's import set is untouched.

Tests in `budgetexempt_test.go`, driven through `Shed.Step` over the package's existing test support:
- an exempt Stuck routes to `OnStuck` and persists `budget_exempt: true` on its history entry;
- a row with `MaxBounces` N survives any number of exempt Stucks interleaved with fewer than N counted ones, and blocks with `ReasonBounceBudgetExhausted` on the (N+1)th counted Stuck;
- a history entry JSON without the field decodes as not exempt and is counted;
- a non-Stuck outcome carrying `BudgetExempt: true` persists no exempt flag.

Add `BenchmarkStep_LongHistory`, stepping a self-bouncing row over a 5000-entry history, so the open risk about long awaits (every exempt bounce rewrites the whole status file) has a measurement to run; it is a benchmark, so `go test` does not run it by default.

**ImpactSummary:** Additive field on two shedengine structs plus one skip in the budget counter; existing status files decode and count unchanged.
