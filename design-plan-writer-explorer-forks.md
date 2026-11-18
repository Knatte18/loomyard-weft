# Bench the explorer subagents of the Discussion and Plan writers: with against without

`plan-writer-explorer-forks` · Task · research, loom, stencil · landing · Finalize

## What was seen

Both writer stencils on main tell the writer to hand exploration and lookups to fresh subagents through the Agent tool:
`contracts/stencils/loom/loom-template-discussion.md` ("Start the exploration with explore subagents", `3085f451`, 2026-11-05; lookups since `2d319fe5`)
and `contracts/stencils/loom/loom-template-plan.md` (a batch of lookup subagents).
These are not the twill-plan forks; twill stays, since it makes the first plan version much better than a solo writer
(operator, 2026-11-18; `docs/research/loom/plan-writer-bench-2026-11-12/report.md`).
`tools/tokencount` counts every subagent transcript as a fork, so its "Plan writer forks" $137.84 (21 tapestry runs since 2026-11-03) mixes twill forks and explorers;
a subagent's meta file tells them apart (`agentType` `fork` against `general-purpose` or `Explore`).
The earlier measurement (`docs/research/turns/writer-lookups-measure-2026-11-04/report.md`) counted how writers use them, never whether they help.
The operator doubts they help much.

## Bench, exact cap

- Reference: one landed tapestry run with both a Discussion and a twill Plan, its records archived on its weft tag; the base commit it started from.
- Arms, each once: Discussion-Write with explorers (stencil as on main) and without (the explorer and lookup lines removed in the bench pair's stencil copy);
  Plan-Write with and without, both from the reference's frozen decision record, twill on in both. Four writer sessions in all, model and effort as the hub default.
- Cost and time: from the session transcripts, with explorer subagents split out by `agentType`.
- Quality: grading keys fixed before any bench output is read, from the reference run's own Discussion and Plan review findings, graded as in the 2026-11-12 plan bench (repeated, partly, avoided).
- Also: the explorer share of "Plan writer forks" and of the Discussion writer over the 21 runs counted, from the meta files.
- Cost cap $40 for the bench sessions.

## Output

A report under `docs/research/loom/writer-explorers-bench-<date>/` with the cost, time and grade per arm, and a recommendation: keep the explorers, drop them, or keep them in one writer only.
