# Fixer report: discussion, round 2

## Fixed

- F1 (MEDIUM, design): checked against `internal/yamlengine/reconcile.go`; `applyExistingSequences` carries an existing list only onto a template sequence, while `carryOpenMaps` carries an open-map key whole whatever its shape.
  D4 now requires `discussion_advisors` to join `loomengine.ConfigOpenMaps`, says why, and its Bound covers the count before and after a reconcile.
  The PATTERN-config-strictness Constraints bullet names the key as an open-map key of the loom module.
  The reconcile Acceptance criterion now requires a `loom.yaml` holding the key as `""`, null, a single model-spec, `[]` or a two-entry list to keep its value through `lyx config reconcile`.
- F2 (MEDIUM, design): checked against `contracts/stencils/orch/orch-template-role.md` ("Investigating a stop" acts on a message from a driver, webster or review loop) and PATTERN-parent-directive, which puts the orch stencils outside the rule.
  D8's Bound now keeps the parent-directive argument for other pairs' spawned sessions only, and states the advisor-to-parent message as not bounded, resting on the orch's judgment alone.
  Added an Open risks entry for it.
- F3 (LOW, design): checked against `internal/seatengine/engine.go`; `join` waits on the chair, whose gate runs inside `StartGated`, and only then calls `stopAdvisors`.
  D5's Bound now says which checks an edit passes depending on when it is made, and the Not bounded sentence names the gate-time window, where the Discussion-Review perch is the only check left.
  The author's call: accept the window, since stopping advisors before the gate would leave no advisor up for a parent-review reject's rework (D9) and needs a seat engine change.
  That rejected option is in the support log's Rejected alternatives, and the window has an Open risks entry.

## Disputed

None.

## Deferred

None.
