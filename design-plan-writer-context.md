# Plan writer cost: sub-planners per part, and twill versus a solo writer

`plan-writer-context` · Note · research, loom, stencil

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
  Still, test a solo Opus high writer: compare twill, solo Opus high and solo Fable on Plan cost, Plan rounds and Plan wall time (`lym report timing`) per run.

## Split writing: two variants to bench (operator, 2026-10-22)

Discussion-Write and Plan-Write already run as `shedadapters.MultiLLMProducer` over a `seatengine` table, a chair and advisors; advisors only advise, and twill is the chair forking itself, so today one session writes the whole product.
Two ways to have parts written separately, both benched against each other and against the twill and solo writers before anything reaches production:

1. **Seats with writing assistants.** The chair splits the scope, fixes the interfaces and shared spots, and hands each assistant seat its part; it then joins and checks the parts.
   A new seat role beside the advisor.
   Risks: assistants see neither each other's work nor a shared context, so seams fall to the chair and review, and overlapping parts make several seats read the same code.
2. **Fork-based.** The chair keeps its context lean (scope, split and shared spots only) and forks once per part, so every part writer inherits the same overview without the writer's full reading.
   Technically still one writer with forks, like twill but forking early.

## Data check (Sonnet agent, 2026-10-24)

Plan writer plus its lens forks, means per run (API list price):

| group | runs | writer+forks | forks | Plan-Write wall | Plan rounds |
|---|---|---|---|---|---|
| before 10-15 (solo writer) | 28 | $9.08 | $0.58 | 16.4 min active | 1.3 |
| 10-15 to 10-19 | 41 | $9.39 | $3.86 | 18.5 min | 1.88 |
| 10-20 to c62274be5 (full twill-plan fan) | 7 | $18.14 | $7.66 | 23.9 min | 2.00 |
| after c62274be5 (five lenses) | 3 | $16.64 | $7.40 | 23.9 min | 1.33 |

- The later groups are larger tasks, so the cost rise is not the fan alone; the five-lens group is too small to judge.
- `relevant.md` use, over 32 runs with a list and a Plan-Write transcript: 85% of named files were read, but only 28% of read files were named (about 3.5 files read per file named, about 7 on large runs). Both shares are lower bounds (directory greps and quarry reads not counted).
- So `relevant.md` works as a floor, not a boundary; most unnamed reading looks like tests and neighbouring callers, which a per-part split of the list would not remove.
- Still open: the solo Opus high and Fable writer bench, and the split-writing variants, which need new runs.
- Scripts, baseline tables and `results.md` are in `docs/research/plan-writer-audit-2026-10-24/`; rerun per its README.

## Discussion versus Plan: double reading (Sonnet agent, 2026-10-25)

55 loom runs seeded 2026-10-15 or later with both a `ly:<slug>:discussion` and a `ly:<slug>:plan` transcript on this machine; cost at API list price from the 10-24 tokencount reports, per run:

| | Discussion | Plan |
|---|---|---|
| phase total (writer, forks, review, fix) | $8.68 | $18.51 |
| writer session alone | $2.07 | $6.60 |
| writer turns / mean context | 39 / 89k tokens | 83 / 183k tokens |
| repo files read by the writer (pooled) | 1,093 | 3,286 |
| share read by both | 62% of Discussion's reads re-read by Plan | 21% of Plan's reads read by Discussion |
| same, without `PATTERN.md`, `CLAUDE.md`, `pattern/`, `docs/` | 64% | 20% |

- Discussion costs 0.47 of Plan for the phase and 0.31 for the writer alone; Discussion was the dearer phase in only 4 of 55 runs.
- Plan-Write re-reads about 62% of the files Discussion-Write read (mean of per-run shares 62%), so exploration is repeated, but Plan reads about three times as many files and most of its reading (79%) is new.
- Rough size of the repeat: the Plan-Write reads that Discussion had already made are about 3.3M tokens pooled, about 59k per run (file bytes at current `main` divided by 4; whole files assumed, so an upper bound).
  That is roughly a third of the Plan writer's mean context, and it is paid again on every later turn through the cache.
- File-level only: directory greps, globs and `lym quarry` reads are not counted, and a file is the same file by path, not by the lines read.
- The Discussion phase total holds its review rounds and fixers, which is most of its cost beyond the writer; the segment figures come from the 10-24 tokencount reports, so runs landed since are priced as of then.
- Rerun: `.scratch/discussion-overlap/reads2.sh <slug> plan|discussion` per run, then `analyze.sh`; both stay in `.scratch` and are not committed.
- Decision (operator, 2026-10-25): Discussion stays a separate step; folding it into the Plan writer would add turns to the costliest session.

## Turn audit (landed 2026-10-28)

`docs/research/turn-audit-2026-10-28/report.md` (n = 1 loom run, 213 sessions) finds the Plan writer spends 33 of its 101 turns on serial single-call reads and searches, mostly shell `grep` and `sed -n`, which a merged turn would remove.
It recommends one stencil sentence in `loom-template-plan.md` and in the twill lens directive: issue every search and read you can name in one message.
This is a cheaper cut than the digest, the earlier fork and the split writing above, and the audit proposes no pre-read for the writer.
The recommendation is untested: no run measured the effect, and the Plan row rests on one session.
