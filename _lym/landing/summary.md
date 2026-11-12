# Report: twill against a solo Opus and a solo Fable on one plan

Adds a research report under `docs/research/loom/plan-writer-bench-2026-11-12/` that answers once whether the twill Plan writer beats a solo writer.
No code, config default or stencil changes.

## What the bench ran

- The reference is `machine-name-verb`, the most recent landed tapestry run with a twill Plan-Write; its base commit is `0604f2ea`.
- Arm A is the landed run itself, with its figures read from the archive tag's usage records.
- Arm B is a solo `opus[high]` writer and arm C a solo `fable[medium]` writer, each run once from the frozen Discussion record on the base commit with `plan_lenses` empty.
- The orch created and ran the two pairs, because `lym fabric add` is refused from a task session; the parent recorded this with `lym loom decision add`.

## What it found

- Solo Opus costs $2.87 and 7m19s a plan against $4.86 and 14m13s for the twill writer, and $12.41 and 41m11s for the twill Plan phase with its two review rounds.
- Solo Fable costs about $6.73 (an estimate, since `lym` recorded no dollars for it) and 9m00s with a 31m19s wait taken out.
- Graded on nine keys drawn from the landed run's own Plan review findings, solo Opus repeats two and partly repeats two, with no blocking key; solo Fable repeats three, both blocking keys among them, and partly repeats two.
- The report states that the keys favour the twill plan by construction, and that one plan from one grader gives a direction only.

## Files

- `keys.md` holds the grading keys, committed before any solo plan was read.
- `report.md` holds the cost and time table, the grading table, the verdict and the deviations.

## Design decisions

- Arm C's cost is priced at the `claude-fable-5-1` list price because `pricing.go` lacks `claude-fable-5-2`; the report says so beside the figure.
- The tokens for arm A come from the tag's usage records because `tokencount` found no Plan transcripts for it on this machine.
- No default and no `plan_split` producer is changed, as the task excludes.
