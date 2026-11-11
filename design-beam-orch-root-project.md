# A beam orch cannot create a project at the root, only under its own beam

`beam-orch-root-project` · Task · feature, board, orch, workspaces · running · Drift-Verify

Imported from [issue #55](https://github.com/Knatte18/loomyard/issues/55).

## What happened

Models hub, 2026-11-04.
The motif `knut-rheology-friction-pvt`, run inside beam `duplication-and-import-followups` (P2), decided (its D1) that the follow-up work runs as a project of its own outside that beam.
The beam's orch, on the operator's order, tried to create that project (`knut-rheology`) at the root and was refused: a beam orch may create entries only under its own beam.
It had to create the project under P2 and the task split under it, and the operator then had to have the prime orch move things.

## Why it matters

A beam is meant to land and disappear. A project a beam's motif spins out often belongs beside the beam, not inside it, and keeping it inside blocks the beam's landing.
The operator gave the order in the beam's orch session, where the discussion happened, and had to repeat it in the prime.

## Decision (operator, 2026-11-06)

The boundary stays: a beam orch writes only under its own beam, since each orch keeps order in its own part of the board.
No flag lets a beam orch write outside it.

Instead the beam orch passes the wish to its parent orch itself, through creel, so the operator gives the order once:

- It goes to the beam's own parent, not to the prime: a nested beam asks its parent, which passes it upward when the work belongs higher up.
- It is a wish, not a finished order: it says what should be made and why.
- The parent decides the placement from what only it sees: its running tasks and their overlap, `depends_on`, whether the work is a project of its own or belongs under an existing one, and how many runs it has.
  It may place the work differently, and it tells the operator when something collides.

## Wanted

- One rule in the orch role saying the above.
  `orch-role-shorter` has landed and rewrote `contracts/orchrole/orch-role.md`, so the rule goes into the shortened role, most likely under `## Messaging` or `## Working rules`; the beam material now sits in `role-beams.md`.
- The refusal names this way forward. Where the refusal sits in the code is still to be found.
