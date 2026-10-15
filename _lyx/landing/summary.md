# Twill: fork0 plan writer, critical judge, branch-coverage lens

Plan-Write with its lens forks on now runs the design the twill benches settled on.
The Plan-Write lead explores and reads the rubric, then spawns one fork, fork0, which writes every card file and the overview and runs `lyx loom validate-plan`.
The lead reads the finished plan in full, without fork0's reasoning, and spawns the lens forks from its own session.
The lenses therefore hold the exploration and the plan but never the writer's reasoning, and the lead judges their findings itself.
`docs/research/twill-fork0-bench.md` and `docs/research/twill-fixer-bench.md` hold the evidence.

## What changes

- **fork0 flow at Plan-Write.**
  It replaces the old writer-then-forks flow whenever `plan_lenses` names a fan; no key selects the old flow.
  Discussion-Write is unchanged and spawns no writer fork.
- **Critical-fixer judge at Plan-Write only.**
  Each finding is weighed twice, its premise and the change it proposes, and gets one of three dispositions: dispute (`premise false` or `every fix worse`, with cited evidence), fix with the judge's own change, or fix as proposed.
  The record lists one line per finding with lens, severity and disposition.
  Discussion-Write keeps today's plain fix-or-reject judge.
- **`twill-plan` fan.**
  New `branch-coverage` lens after `criteria-coverage`: every branch and exit the cards' Intents introduce, named by the record or not, must be reached by a planned test row.
  `forced-edits` gains a sentence that a new type, function or executor needing registration in a table or list is a forced edit.
  `criteria-coverage` gains a sentence to check that a path, comparison or default uses the operand the record names, not a near one such as the wrong parent directory.

## Design decisions

- The twill directive in `internal/twilldirective` gains parts, all host-neutral stencils read through `stencilstore`: `twill-writer-fork` (the fork0 procedure, its failure paths and fork0's prompt), `twill-relaunch`, and one of `twill-judge-critical` or `twill-judge-plain`, chosen per host.
  Host-specific nouns arrive as host values; the renderer errors naming the marker when a required one is empty.
- The failure paths for a session that ends or is resumed are written out as separate rules, using the overview-last completion signal rather than new state.
  Each master session spawns fork0 at most once and lens forks at most once.
  A re-prompted master spawns no fork0 and edits the plan itself.
  This replaces the old "never spawns forks again" sentence at both hosts.
- The fork boilerplate no longer says the writer wrote the files; it says they were written and passed the self-check, which holds for both hosts.
- With lens forks off, the rendered prompt is unchanged, and the Plan-Write gate still requires a recorded `lens-review.md`.
- The `rejection_rule` host value is retired, and the directive's closed marker list in `contracts/stencils/markers_test.go` changes with it.

## Docs and tests

The `twilldirective` package doc, `docs/overview.md` and the `plan_lenses` comment in `internal/loomengine/template.yaml` describe the new flow.
Claims, markers, directive and fan tests are extended for the new parts, host-neutrality, the judge split, and the fan's lens list.

## To check by hand

- A hub `burler.yaml` that overrides the `twill-plan` fan keeps its own lens list and misses `branch-coverage` until edited.
- Only Plan-Write was benched with fork0 and the critical judge; the Discussion-Write plain-judge text is unchanged.
- No real Plan-Write run with fork0 was observed end to end, including the resume and re-prompt paths; run one with lens forks on and check that `lens-review.md` is recorded and the gate passes.
- Read the `Directive` renderer changes in `internal/twilldirective/twilldirective.go`, which fill exactly one judge part into a single judge marker.
