---
round: 2
exclude_lenses: []
focus:
  - "Runner liveness: in decision-record.md D1 (join rule) and D3 (lease), settle whether a front can still join or renew a run whose runner process was killed by SIGTERM, SIGKILL (internal/runproc/end.go ends with SIGTERM then SIGKILL after 5 s) or an OOM kill, and what bounds that path."
  - "Lease semantics: in D3 and D5, settle whether a front with a shorter cap joining a run a longer-cap front waits on can move the lease earlier, and whether the lease write is ordered before the runner's spawn."
  - "Median inputs: in D5, settle whether gateslot.RunRecord (internal/gateslot/runlog.go) has any field marking an abandoned call, given that execution.report in internal/gatecli/run.go writes a cancelled tier's record, and which record times form a call's wall time."
  - "Card-verify path: in D6, settle what a card-verify runner does for a Verify line that is a --wait gate call versus any other line, given internal/cardverify/runner.go runs the former unslotted as a child under context.Background."
  - "Runner verb bound: in D1, settle what a direct call of the hidden runner verb can do and what limits it, with the refusal naming its way forward."
---

All lenses ran in round 1 with MEDIUM or worse findings except none settled, so none is excluded.
The focus entries name the areas the round-1 design fixes will rewrite; new text there has had no fresh review.
