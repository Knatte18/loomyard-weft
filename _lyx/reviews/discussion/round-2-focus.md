---
round: 2
focus:
  - "D6 in decision-record.md: webster resolves two role engines, Master and the recovery strand, and `loadRunWrites` (`internal/websterengine/runwrites.go`) fork-audits every recorded recovery session, read by `reset` and `recover-batch`. Settle whether each role's engine is checked for fork audit before Master spawns, whether each refusal names its own config key, and whether the Bound states every guard that is relaxed."
  - "D6 in decision-record.md: it decides that a change to the `master` key does not change the engine an existing run is read through. Settle, for each refusal check point (wiring and lookup of a recorded engine), whether its stated way forward can actually clear that refusal, per `PATTERN-refusal-way-forward`, including the case where the run record carries no engine and reads as the current `shuttle.yaml` default."
  - "D8 in decision-record.md: the `Requires` refusal fires at `Start`. Settle whether every caller that can reach it has a key or a stated alternative to name, in particular `lyx burler run` with a cluster-fan profile (`internal/burlercli/run.go`), whose reviewer model comes from flags with an empty engine."
  - "D2 in decision-record.md and the `PATTERN-model-choice-mapping` scan it pins: D2's rationale is that hand mappings are where the engine was dropped. Settle whether the scan fails on a new site that copies `modelspec.Resolved.Model` and drops `Engine`, or states a bound for that bypass."
  - "D5 in decision-record.md against Scope's Out rule that a role is not marked `peer-messaging` for its parent asks: settle whether every role that messages its parent (the loom driver's notify stencils, Discussion-Write's delivery prompt) is classified the same way."
---

The entries target the design questions open at the end of round 1.
Acceptance-criterion wording and the Auto-mode assumptions line are checked by the usual rubric pass and need no focus entry.
