# Support log — teardown-retry-no-pair

## Interview

A darn run holds no interview; the board entry is the whole statement of the change.

## Rejected alternatives

- Let `fabricengine.ReconcileOrigin`'s run scope accept a slug with no pair worktree: it would widen the one origin judge for every caller to fix one resume, and the scope would still have no branch of the run to judge.
- Treat the skipped reconcile as an unreachable origin, writing the origin-owed marker: the marker would ask for a reconcile the run can never run again.
- Let `lym batten cancel` end a landed run blocked at Worktree-Teardown: cancel's refusal already names `lym batten run`, and once that works no second route is needed.
- Teach `lym batten cleanup` to remove blocked records: it would drop a run whose remote branch is still left without deleting it.

## Review rounds

_No rounds yet._

## Question ledger

- Should the skip also cover other producers whose pair is gone? Self-picked no: only Worktree-Teardown expects a gone pair; any other producer with its pair gone is a fault the reconcile's refusal should keep surfacing.
- Should the owner still be judged when the reconcile is skipped? Self-picked yes, against the local board, as an unreachable origin is.
- Should the refusal for another owner keep "re-run once origin answers"? Self-picked no for this case (D3).
