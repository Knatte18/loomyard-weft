# Plan delete-order check reports a reference the deleting card itself rewrites

`delete-order-own-edit` · Task · bug, webster, loom · running · Darn-Burler-Review

## Seen

`overview-seats` is blocked at Webster: begin-batch 17 is refused with `plan_drifted`, `delete-before-reference/18-retired-builders-delete[blocking]`: card 18 deletes `internal/loomengine#PlanSpec`, but card 19 `19-loom-config-axes` still references it.

- The reference is `internal/loomengine/plan_test.go:79`, inside `TestPlanTable`, a temporary parity test that card 12's batch added (`fbc22e3ff`) after Plan-Write validated the plan.
- Card 18 lists `internal/loomengine#TestPlanTable` under Edit itself, and its Intent drops the byte comparison with the old spec builder, which removes that call before card 19 runs.
- `deleteOrderFindings` (`internal/planglyph/deleteorder.go`) only asks whether a later card's Edit region holds the reference; it does not exempt a reference that sits inside a region the deleting card, or an earlier card, edits itself.
- The begin-batch drift check reads the current tree, so a reference added by an earlier batch shows up only then; begin-batch has no override.

## Fix

- `deleteOrderFindings` skips a reference that sits inside an Edit region of the deleting card or of any card before it, since that card rewrites the region before the later card runs.
- A test pins the case: a reference that a later batch's region holds but the deleting card's own Edit region also covers is not reported; a reference only a later card covers still is.
- Docs: `internal/planglyph` doc.go if it states the rule.

## After it lands

The orch resumes `overview-seats` with `lyx loom resume` in its pair.
