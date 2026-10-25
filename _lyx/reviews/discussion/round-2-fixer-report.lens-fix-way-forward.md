```yaml
lens: fix-way-forward
outcome: reported
```

Task: check that every way forward the fix introduced can be carried out from the state it names and leads to progress.

- **MEDIUM:** loom's Windows foreground fallback Stuck names no way forward.
  - Changed text: decision record D4, Windows paragraph: "When a job lyx did not create still refuses the breakaway, the row runs the capture in the foreground as darn does, returning Done on a passed capture and Stuck on a failed one".
  - Evidence: D3's added sentence gives a way forward only to "Darn's Stuck on a failed capture" (`lyx drift capture` or `lyx drift defer`, each followed by resume).
  - D5's widened `lyx drift defer` admits "`Drift-Baseline` on a failed or refused capture", so defer would work there, but no halt text names it for loom.
  - That breaks the Constraint "Every refusal and halt names its way forward and a test reaches each", and no acceptance criterion covers this Stuck.
- **LOW:** `lyx drift defer` at `Drift-Baseline` has no mechanism to tell a refused capture from other halts.
  - Changed text: D5: "`lyx drift defer` … refuses unless the run is halted at `Drift-Await`, or at `Drift-Baseline` on a failed or refused capture (D3)".
  - Evidence: a capture D4 refuses never starts, so it writes no capture record (D4: "a fresh capture writes a fresh record" only for a capture that runs).
  - So the verb cannot tell a refused-capture halt from another `Drift-Baseline` halt, such as a scope agent's failure, by the capture record.
  - The record names no mechanism, for example the halt reason, so implementing this refusal is left unstated.
- **LOW:** D4 still describes only the `Drift-Await` caller.
  - Changed text: D3: "Darn's Stuck on a failed capture names `lyx drift capture` followed by `lyx loom resume`".
  - Evidence: D4 says "Run by a caller at a `Drift-Await` halt, `lyx drift capture` starts the same detached, registered child". D4's main-capture refusal admits `Drift-Baseline` as the current row, so the step can be carried out.
  - But the caller's behaviour at a darn `Drift-Baseline` halt, a detached child that the resumed row then waits on, is stated only through D3's "A capture holding the lock is waited on in darn". D4's caller sentence still names only `Drift-Await`.

Every other way forward I checked can be carried out and leads to progress:
- `lyx drift waive` followed by resume, from the no-passed-capture halt, including after adopt and under scope `full` by the parent.
- Darn's defer, then resume, then Done, then the `Drift-Verify` waive.
- The refused-capture Stuck in loom, defer, then the `awaited_at` Done.
- A caller's `lyx drift accept` at decision halts.
- The `partial` withdrawn-list decision halt, which the parent can lift.
- The new acceptance criteria are all reachable.

Further tool calls: none.
