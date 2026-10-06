---
round: 3
focus:
  - "D7 in `_lyx/discussion/decision-record.md`: for a task branch whose local HEAD was already rewound while `origin/<task-branch>` still holds the run's own abandoned commits, settle whether the stated way forward converges without a conflict-prone merge, who resolves a conflict, and what separates the run's own commits from someone else's pushed ones; check the acceptance criteria cover that state."
  - "D7 order of steps: settle whether the remote update runs before or after `ResetPairWarp`'s ownership and dirtiness checks, what message and way forward a local refusal carries after the remote already moved, what a network or auth failure of the remote update leaves and names, and whether each has a `refusal-spec.md` row; also whether a remote tip that is an ancestor of the reset commit is skipped rather than pushed forward."
  - "D6 in `_lyx/discussion/decision-record.md`: settle whether the premise for restamping a failed, dead or stuck batch matches `internal/websterengine/recoverbatch.go`, which keeps a dead-classified strand alive and stops the prior live strand before the recovery spawn; check the acceptance criterion names every terminal status the Decision accepts."
  - "D4 and D6 way forward: settle whether naming `recover-batch NN` holds for a batch whose record carries uncheckable paths, given that `RecoverSpawnOrAttach` refuses such a failed batch and names `run --fresh`, and whether the order of the two refusals is stated."
  - "D3 Bound: settle whether the stated bound on a passing missing `Delete` target matches what `DoneChecks` does with a `not_found` state for a Delete target, and what actually bounds a target the tree never held."
  - "D1 to D4 card scope: settle whether the scope phrase matches `DispatchScope` in `internal/websterengine/beginbatch.go` (which drops cards of begun batches and keeps a forthcoming subset), whether D4 says which later cards it compares, and whether D1's claim to mirror `deriveEdges` holds given `deriveEdges` draws edges only between cards of different batches."
  - "D10 in `_lyx/discussion/decision-record.md`: settle whether the Decision states what removing the model assertion lets through and what bounds it, and whether its rationale describes the check the code actually performs."
---

Each entry names a site and the question to settle; none caps severity or states a verdict.
