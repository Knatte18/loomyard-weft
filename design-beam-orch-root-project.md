# A beam orch cannot create a project at the root, only under its own beam

`beam-orch-root-project` · Note · feature, board, orch, workspaces

Imported from [issue #55](https://github.com/Knatte18/loomyard/issues/55).

## What happened

Models hub, 2026-11-04.
The motif `knut-rheology-friction-pvt`, run inside beam `duplication-and-import-followups` (P2), decided (its D1) that the follow-up work runs as a project of its own outside that beam.
The beam's orch, on the operator's order, tried to create that project (`knut-rheology`) at the root and was refused: a beam orch may create entries only under its own beam.
It had to create the project under P2 and the task split under it, and the operator then had to have the prime orch move things.

## Why it matters

A beam is meant to land and disappear. A project a beam's motif spins out often belongs beside the beam, not inside it, and keeping it inside blocks the beam's landing.
The operator gave the order in the beam's orch session, where the discussion happened, and had to repeat it in the prime.

## Expected

An operator-ordered board write from a beam orch can target the root (or another project), e.g. with an explicit flag or owner field, or lym names the way: "ask the prime orch" with the exact payload to post.