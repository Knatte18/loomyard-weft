# Plan writer context grows to 300k and every lens fork inherits it

`plan-writer-context` · Note · enhancement, loom, stencil

## Seen

`docs/research/token-audit-2026-10-22`: the Plan writer is the costliest session type, about $8 per run, $1,306 with its forks over two weeks (18% of all spend).
It averages 81 turns at a mean context of 223k, holds 153k by its first fork and peaks around 308k, mostly from about 71 raw `sed -n`/`grep` code reads.
Every `twill-plan` lens fork inherits that context, about 287k per turn; since 2026-10-20 the writer and its forks cost about $18 per run.

## Idea (orch, not decided)

Fork the lenses earlier, before the code reads pile up, or hand the writer a digest of the code it needs instead of raw reads.
Cutting a quarter of the writer's and its forks' cost saves about 4.5% of all spend; quality must be benched first.
