---
round: 4
focus:
  - "Settle whether every path that moves a checked-out weft of a pair or the prime is stopped before it changes a checked-out marker or while a D7 move record stands: look in `_lyx/discussion/decision-record.md` at D10's pull, push and checkout stops, D7 step 1's list of refusers, and the Acceptance criteria, and compare against the origin reconcile's moving half in `internal/fabricengine/originreconcilemoves.go` (`reconcileRecordsBranch`, `ffWorktree`, `replayWeft`, `moveAll`, `movePrime`), run hub-wide by `lyx orch start`, `lyx adopt`, `lyx hibernate` and `lyx cleanup` and run-scoped on a run's own pair. The question is whether D10's rationale claim that each path refuses or stops before changing anything holds, and what each stop leaves unchanged."
  - "Settle whether each refusal in D5, D7, D8 and D10 names a way forward the operator can run: check the checkout refusal's `lyx fabric anchor --set <target anchor>` against D7's validation that the anchor is a directory tracked in the worktree's current warp checkout, and check the move-lock refusal's named pid against what the move record in D7 step 1 holds."
  - "Settle whether the markerless value D5 defines covers every read of a marker in the decision record, including the marker of a replayed local commit's parent tree in D10's integration paragraph, and whether the Acceptance criteria for `lyx fabric anchor --set` state each guard that D7 step 1 relies on, including a live dobby starting no pass and no task while the move record stands."
---

Round 4 follows a round whose single BLOCKING finding concerned a weft-moving path outside D10's stop.
The entries above name where to look and the question to settle, and cap nothing.
