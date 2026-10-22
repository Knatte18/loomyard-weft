# Plan writer cost: sub-planners per part, and twill versus a solo writer

`plan-writer-context` · Note · enhancement, loom, stencil

## Seen

`docs/research/token-audit-2026-10-22`: the Plan writer is the costliest session type, about $8 per run, $1,306 with its forks over two weeks (18% of all spend).
It averages 81 turns at a mean context of 223k, holds 153k by its first fork and peaks around 308k, mostly from about 71 raw `sed -n`/`grep` code reads.
Every `twill-plan` lens fork inherits that context, about 287k per turn; since 2026-10-20 the writer and its forks cost about $18 per run.

## Idea (orch, not decided)

Fork the lenses earlier, before the code reads pile up, or hand the writer a digest of the code it needs instead of raw reads.
Cutting a quarter of the writer's and its forks' cost saves about 4.5% of all spend; quality must be benched first.

## Discussion with the operator (2026-10-22)

- A code digest instead of raw reads is rejected: it is too inexact unless the code itself is read, and the writer would read the source anyway to be sure.
  Finding what to read is already `relevant.md`'s job; whether the writer uses it is worth measuring (how much of what the writer read the list named).
- Lens forks as fresh agents are rejected: that is the perch itself.
- Sub-planners: Discussion splits the scope into parts in the decision record, groups `relevant.md` per part, and each sub-planner reads only its part; the lead owns the shared spots and joins the card DAG.
  This keeps every review round and matters more as loom tasks are bundled larger.
  Review could split per part the same way, since each fresh reviewer rebuilds its context every round (Plan reviewer: 48 turns at 123k mean context, $2.48 per session).
- Twill-plan versus a solo writer: since the full `twill-plan` fan (2026-10-20), 7 landed loom runs average 2.00 Plan rounds against 2.09 before, while the writer's forks cost about $7 per run and a Plan round about $3.70; too few runs to judge.
  The operator sees strong signs that the twill writer makes a far better one-shot plan in shorter time, and time counts as well as tokens.
  Still, test a solo Opus high writer: compare twill, solo Opus high and solo Fable on Plan cost, Plan rounds and Plan wall time (`lyx report timing`) per run.

## Split writing: two variants to bench (operator, 2026-10-22)

Discussion-Write and Plan-Write already run as `shedadapters.MultiLLMProducer` over a `seatengine` table, a chair and advisors; advisors only advise, and twill is the chair forking itself, so today one session writes the whole product.
Two ways to have parts written separately, both benched against each other and against the twill and solo writers before anything reaches production:

1. **Seats with writing assistants.** The chair splits the scope, fixes the interfaces and shared spots, and hands each assistant seat its part; it then joins and checks the parts.
   A new seat role beside the advisor.
   Risks: assistants see neither each other's work nor a shared context, so seams fall to the chair and review, and overlapping parts make several seats read the same code.
2. **Fork-based.** The chair keeps its context lean (scope, split and shared spots only) and forks once per part, so every part writer inherits the same overview without the writer's full reading.
   Technically still one writer with forks, like twill but forking early.
