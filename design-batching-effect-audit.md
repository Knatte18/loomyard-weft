# Measure what the batching sentences did to the Plan and Discussion writers

`batching-effect-audit` · Task · research, stencil, shuttle, loom · running · Darn-Write

## Goal

Measure what the batching sentences did to the costliest sessions.
Since `bf00a043` (2026-10-28) the stencils and shuttle's appended system prompt tell every agent to issue each call whose arguments it knows in one message.
`docs/research/turn-audit-2026-10-28/report.md` recommended them for the Plan writer, which spent 33 of 101 turns on serial single reads and searches, and no run has measured their effect.
The answer decides how much the plan-writer ideas of notes `plan-writer-context` and `loom-beam-darns` are still worth.

## Scope

- Plan-Write writer sessions of loom runs landed before and after 2026-10-28, every one with a transcript on Neve.
- The same for the Discussion-Write writer sessions, to see whether the effect is general.
- Per session and as means per group: turns, tool calls per turn, serial single-call read and search turns, mean and peak context, and cost at API list price.
- Note task size beside each run, since the later tasks are larger, and say how far the groups can be compared.

## Cap

No new runs and no agent runs: analysis of existing transcripts only.
Reuse the scripts of `docs/research/plan-writer-audit-2026-10-24/` and `docs/research/turn-audit-2026-10-28/` where they fit.
The report goes to `docs/research/` with the scripts beside it and a README that says how to rerun it.
