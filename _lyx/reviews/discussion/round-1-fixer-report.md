# Fixer report — discussion round 1

## Changes

- F1 (fixed as proposed): D6's wiring check covers the engines of both Master's and the recovery strand's `ModelChoice` before Master spawns, each refusal naming `master` or `recovery` in `webster.yaml`.
  The Bound now states both role engines are checked before spawn and the default engine is the only one no longer checked.
  New acceptance criterion: a recovery choice on an engine without fork audit is refused before Master spawns.
- F2 (fixed as proposed): D6 splits the two ways forward.
  The lookup refusal states its cause, a record with no engine value read as the current default, and its way forward: set `engine:` in `shuttle.yaml` back to an engine with fork audit, then re-run the verb or re-step the Webster row.
  A recorded engine value without fork audit is an internal error, not a refusal row.
  New acceptance criterion for the absent-record case.
- F3 (fixed differently): D8 names the `lyx burler run` caller, whose refusal names the default engine with the way forward to set `engine:` in `shuttle.yaml`, then re-run the verb, and adds the proposed acceptance criterion.
  Not made: the refusal-spec row "in the burler section".
  `contracts/specs/refusal-spec.md` has no burler section, and its Out of scope section says "Landing, batten, orch, fabric, burler and board refusals are not in this table", so D8 states that this refusal lands no row.
- F4 (fixed as proposed): the driver row drops `peer-messaging`, the rule loses its parenthetical about the driver's notify stencils, and the `creel-agents` Open risk names the seat rows only.
- F5 (fixed as proposed): D2's rule and the first acceptance criterion cover reads of any `modelspec.Resolved` field (`Engine`, `Model` or `Params`).
  `PATTERN-model-choice-mapping` is named by D2 with that wording, so the entry carries it.
- F6 (fixed as proposed): the criterion reads "omits the key for a half or pass with no run record".
- F7 (fixed as proposed): the pointer to the support log's ledger is gone.
- Support log: a Round 1 entry under Review rounds.

## Disputed

None.

## Deferred

None.
