# Round 1 fixer report

All six findings fixed in `_lyx/discussion/decision-record.md`; `support-log.md` gained ledger entries for the new picks.

## Changed

- F1: D4 now commits and pushes the seeded or reconciled hub-wide files in `_board` through `fabricengine.Bolt` under the board write lock; a push failure is reported and left for the next board sync.
  The Bound covers only the config files the reconcile wrote.
- F2: D6 deletes a per-worktree copy only when the hub file holds everything it holds; otherwise it reports the copy as divergent and keeps it.
  The Bound and the `lyx config reconcile` acceptance bullet say so.
- F3: D5 validates the edit through the strict load before commit, does the write and commit under the write lock, and states the failed-commit outcome (file stays, error says uncommitted, next board sync commits it).
  The Bound and a new acceptance bullet carry it.
- F4: the seed-source report moved into D4 and the fourth acceptance bullet; the Open risks bullet is trimmed to the residual risk.
- F5: D4 now decides the no-prime case: template seed, silent in `lyx fabric clone`, with a warning in `lyx fabric reconcile`.
  The prime premise and its two failure modes (no resolvable prime, lagging copy) are recorded under Auto-mode assumptions.
- F6: the `boardengine/doc.go` paragraph bullet left Auto-mode assumptions; the Constraints docs line now carries the requirement.
- Support log: Q9 to Q11 added to the question ledger.

## Deferred

None.
