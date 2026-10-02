# Fixer report — discussion round 1

Every finding in `round-1-review.md` is fixed; `lyx loom validate-discussion` exits 0 afterwards.

## Changes

- **F1 (BLOCKING):** Decision 5 now makes the round store decide the cap.
  `attempts: 3` is told to the gate as the cap.
  A reject below the cap re-prompts.
  The cap's reject returns a new terminal failure on `GateResult` that the engine finalizes without a re-prompt.
  Because the in-memory count never exceeds the store count, the engine's budget cannot fail the entry first.
- **F2 (BLOCKING):** Decision 5 keys the lifted `Pending`/`Final` guard on a new `GateEntry` field that only the parent-review wiring sets.
  Its Bound states what the entry passes on.
  `Final` now fails on an unconsumed reject and counts it toward the cap.
  This replaces today's Warn-and-pass and is recorded under Auto-mode assumptions, in the support log's Rejected alternatives and as ledger Q11.
- **F3 (BLOCKING):** Decision 5 now states how the fresh-spawn round and the gate interact.
  The fresh-spawn preparation keeps the latest round when its verdict is a reject or a superseding approve.
  The gate reads the latest round before opening a request.
  The way out and the halt-again path follow from these two rules.
  A Notes line points at `entries_discussionwrite.go`.
- **F4 (MEDIUM):** Decision 5's Bound now defines "halted at the cap" by three conditions: status `blocked`, cap rejected rounds in the store, and the cap's reject as the latest verdict.
  It also states that approve checks no caller identity.
- **F5 (MEDIUM):** A new acceptance criterion covers the refusals: supersede outside the cap, resume without approval, and session loss after a reject.
  The gate criterion now covers an attach between rejects, and the hold criterion covers a failing liveness read.
- **F6 (MEDIUM):** Decision 4 now treats a failed liveness read as not live: the gate holds with a Warn and never returns a gate error.
- **F7 (MEDIUM):** The sixth constraint is narrowed to the orch session, and adds that batten sends into no session.
- **F8 (LOW):** Decision 4 gains a Bound stating that a reviewer which never becomes live, as for a nested pair, always ends in the timed pass.
  The Open risks item keeps only the upward-message consequence.
- **F9 (LOW):** The `creel` clause is cut from Decision 4's rationale, and the pane-typing argument from Decision 7's.
  Both arguments already sit in the support log's Rejected alternatives, so the support log needed no move.
- **F10 (NIT):** Decision 5's rationale drops the comparison with the `discussion` gate and says three rounds give the writer two rewrites.

## Deferred

None.
