# Begin-batch refusal route and label; bare lyx config writes an empty layer

`refusal-route-config-empty` · Task · bug, webster, loom, config · running · Plan-Write

Two small bugs in one run, bundled on the operator's word (2026-10-15) to save a run's overhead, though they touch different packages.

# Part 1: begin-batch refusal route (issue #625, webster and loom)

## What happened

A task's run passed Plan-Bouncer (done) and Batchifier (done), then Webster's `lyx webster begin-batch 17` refused with `plan_drifted`.
The blocking finding was `delete-before-reference/18-retired-builders-delete[blocking]`: a card deletes a symbol that a later card still references in its Edit code.
`begin-batch 17` was refused twice with the identical finding, and the run halted `blocked` at Webster with anomaly `escalation-to-human`.

## Correction (orch, 2026-10-15)

In `overview-seats` the plan was not malformed: the reference was a temporary parity test that card 12's batch added after Plan-Write, inside a region card 18 itself rewrites, so the finding was a false positive of the check, fixed by `delete-order-own-edit` (landed 2026-10-14).

## What to do

- The refusal is labelled `plan_drifted` even when the plan did not drift from the tree but was malformed at authoring: distinguish the two in the refusal's classification, so the label names the right cause.
- A real `delete-before-reference` refusal at `begin-batch` has no route back to a plan-editing agent and halts for a human, though its text already names the way forward (move the delete to a later card, `lyx webster rebaseline --card NN`, then `begin-batch` again).
  Route it to a plan-fix step, or let Webster or the driver take that way, instead of halting.
- Consider running the `delete-before-reference` check at Plan-Bouncer or Batchifier time, so an ordering defect is fixed by the plan reviewers.

Provenance: task `overview-seats`.

# Part 2: bare `lyx config <module>` writes an empty layer (issue #632, config)

Seen (2026-10-15): a Discussion-Write agent in pair `webster-pack-end-of-prompt` ran `lyx config webster` with no arguments, expecting a read.
It replied `edited and synced _lyx/config/webster.yaml` and left a 0-byte `_lyx/config/webster.yaml` in the pair's worktree layer (behind the weft symlink, so `git status` shows nothing).
The bare form opens an editor; with no interactive terminal the editor exits at once and lyx saves the empty buffer as the layer.

Workaround: the orch never runs `lyx config` without `--set` or `--print`.

Expected: with no TTY, a bare `lyx config <module>` refuses (or prints) and never writes; an empty editor result is never saved over a layer.
