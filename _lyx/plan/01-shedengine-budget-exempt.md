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
- `Shed.stepLocked`'s Stuck case applies its budget arm (`episodeStuckCount(st.History, def.Name) >= effectiveMaxBounces(...)`) to a counted Stuck only: an exempt Stuck skips the comparison.
  Skipping the count alone is not enough, because the arm runs on every Stuck before routing, so after exactly N counted Stucks (all allowed) the first exempt Stuck would read N >= N and block, which is the "blocked by the running budget" case the decision record rules out.
  Routing is otherwise unchanged: an exempt Stuck still follows `OnStuck`, and a row with no `OnStuck` still blocks on it.

The field is plain data on both structs, so the Shed Producer-Seam Invariant's import set is untouched.

Tests in `budgetexempt_test.go`, driven through `Shed.Step` over the package's existing test support:
- an exempt Stuck routes to `OnStuck` and persists `budget_exempt: true` on its history entry;
- a row with `MaxBounces` N survives any number of exempt Stucks interleaved with fewer than N counted ones, and blocks with `ReasonBounceBudgetExhausted` on the (N+1)th counted Stuck;
- a row with `MaxBounces` N whose episode already holds N counted Stucks (the budget fully spent) routes an exempt Stuck to `OnStuck` rather than blocking, and still blocks on the next counted Stuck;
- a history entry JSON without the field decodes as not exempt and is counted;
- a non-Stuck outcome carrying `BudgetExempt: true` persists no exempt flag.

Add `BenchmarkStep_LongHistory`, stepping a self-bouncing row over a 5000-entry history, so the open risk about long awaits (every exempt bounce rewrites the whole status file) has a measurement to run; it is a benchmark, so `go test` does not run it by default.
The same risk's rendering half needs no change: batten's status envelope already bounds the history it reports to `maxStatusHistoryEntries` alongside the true `history_length` (`internal/battencli/arm.go`), and the `progress` bounce count reads `episodeStuckCount`, so it stays at the counted total however many exempt entries accumulate.

**ImpactSummary:** Additive field on two shedengine structs; episodeStuckCount's two callers, the budget arm in run.go and Routing.Bounces behind the status envelope's progress count, both stop counting exempt Stucks, and the budget arm no longer blocks an exempt one; existing status files decode and count unchanged.
