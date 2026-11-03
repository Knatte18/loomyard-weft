# Bench Plan and Discussion fixers on Sonnet against Opus

Adds `docs/research/fixers-on-sonnet-2026-11-03/`: a report and the per-run scores and fixer reports of eight fixer runs, with a recommendation for `plan_fixer_model` and `discussion_fixer_model`.
No code, config or stencil changes.

## What the bench found

- Plan: `sonnet[medium]` fixed all 9 findings in both runs, as `opus[medium]` did, with 2 new defects across its runs against 3 for Opus.
  The report recommends `plan_fixer_model` set to `sonnet[medium]`.
- Discussion: Opus fixed 28 of 30 findings and Sonnet 25, with 7 and 6 new defects.
  The gap is three findings on two runs per arm, so the report recommends leaving `discussion_fixer_model` empty, falling back to `fixer_model`, until a larger bench.
- Nothing was set in any config: the recommendation is the report's, for the operator to apply.

## Design decisions

- The entry's third arm (Sonnet on findings "in the concrete form") was dropped, on the parent's decision recorded after Discussion: `docs/research/concrete-review-findings-2026-11-03/report.md` found every finding already states its place and change, so the arm had no input distinct from the second.
  That leaves eight fixer runs, not twelve, on the two rounds that report keeps in `data/fixers-input.tsv`, minus discussion F3, which offers two alternatives.
- The scoring is a solo `opus[medium]` reviewer with one prompt for every run, as the parent decided, since the hub's fork-fan reviewer cannot be spawned from a darn session.
  It judges each original finding `fixed`, `partial` or `open` and lists the defects the fixer's edits introduced.
- The fixers ran as in-session subagents, not through the repository's reed launch path, with `lym` commands barred and the code read only through `git show` at the round's code commit.
  No `lym` check ran on the fixed artifacts.

## Limits a reviewer should know

- Cost is the subagents' reported tokens and wall time; no prices were available, so the report records no dollar figure.
- One reviewer scored every run and was not blind to the arm, since the run directories name the model.
- The Discussion recommendation rests on a three-finding gap inside two repeats per arm.

## Review focus

- `docs/research/fixers-on-sonnet-2026-11-03/report.md`: whether its tables agree with the `score.md` files under `data/`, and whether the two recommendations follow from them.
