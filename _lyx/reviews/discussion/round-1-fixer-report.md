# Fixer report — discussion round 1

All six findings fixed; `lyx loom validate-discussion` exits 0 on the result.

## Changes

- **F1 (MEDIUM), half-broken-pair fallback.**
  `decision-record.md`: the probe decision now says a pair the probe cannot open, or whose record location cannot be resolved, is a probe error, never "not mid-merge";
  the probe-error decision names both cases, and its acceptance criterion now covers an unopenable pair.
  `support-log.md`: the half-broken-pair rule is added to Rejected alternatives with its reasoning.
- **F2 (LOW), `Fabric.MergeInProgress` rationale.**
  Dropped "needs an open `*Fabric`" from the decision record's rationale and from support-log Interview item 5;
  the probe decision now says the probe opens the pair from `l` itself.
- **F3 (LOW), batten handling.**
  The verified fact moved into the kind decision's rationale;
  the batten auto-mode assumption and the batten open risk are deleted;
  support-log Interview item 8 and ledger Q7 are resolved with the evidence, and the plan-writer note names `innerrun.go`'s retry condition.
- **F4 (LOW), precedence against `driver_not_parked`.**
  The trigger decision now states that a live strand with no marker over a run halted at a hand-back keeps its `driver_not_parked` refusal on a mid-merge pair, with a matching acceptance criterion.
- **F5 (LOW), conflicted-path form.**
  The probe decision now requires the one-repo form a fabric merge verb's `conflicts` carries, with a matching acceptance criterion.
- **F6 (NIT), Goal wording.**
  Reworded to "Today `start` checks no merge state at all before it spawns or resumes a driver".

## Deferred

None.
