# Align each perch writer twill lenses with its reviewer lenses, Discussion first

`writer-reviewer-lens-align` · Note · enhancement, burler, loom, config

## Seen (2026-10-18)

The audit `twill-writer-quality-audit` found that the twill Plan writer halved round-1 Plan findings, while the twill Discussion writer cut round-1 Discussion findings only about 20%, with the same kind of finding left: design holes such as unstated bounds and false premises.

The two sides of the Discussion perch use different kinds of lens (`internal/burlerengine/template.yaml`, fans `twill-discussion` and `review-discussion`; the hub `burler.yaml` overrides neither):

- The writer's twill fan, `twill-discussion`, holds emphasis lenses that say what to look at: `premises`, `goal-scope`, `board-fit`, `pattern-fit`, `scenarios`, `leanness`.
- The reviewer's fan, `review-discussion`, holds procedure lenses that each hunt design holes a set way and end in a list of what they checked: `interactions` (every pair of decisions on the same state, run against each other), `existing-paths` (every existing code path that writes or reads what a decision changes), `bound-breaker` (a concrete counterexample to every bound and claim), `walkthrough` (each mechanism run step by step on the data it is handed).

So only the reviewer hunts design holes by procedure, and design holes are what the reviewer still finds.
At Plan the writer's fan, `twill-plan`, is made of procedure lenses (`criteria-coverage`, `branch-coverage`, `row-reachability`, `forced-edits`, `test-economy`), which target the finding kinds that fell.

## Idea

The writer's and the reviewer's lenses at a perch are aligned: the writer's twill fan runs the procedures the reviewer will judge it by, so the writer finds its own holes before the review.

- At Discussion, `twill-discussion` gains `interactions`, `existing-paths`, `bound-breaker` and `walkthrough`, and keeps the emphasis lenses that cover what those do not, such as `goal-scope`, `board-fit` and `leanness`; whether `premises` and `scenarios` stay beside `bound-breaker` and `existing-paths` is a choice to make.
- At Plan, check `review-plan` against `twill-plan` the same way.
- It is config only: a fan in the hub's `burler.yaml` and `discussion_lenses` naming it, so it can be tried without code.
- It costs more forks per Discussion-Write; weigh that against the review rounds and rework it saves.

## Orch recommendation, not a decision

Try the aligned fan on the next three to five loom runs and compare their round-1 Discussion findings, per run and per card, with the twill baseline in `twill-writer-quality-audit`, under the same review fan.

## Operator decision (2026-10-19): Discussion trial live

The hub `burler.yaml` overrides `twill-discussion` with seven lenses: `interactions`, `existing-paths`, `bound-breaker`, `walkthrough`, `goal-scope`, `board-fit`, `leanness` (Bolt commit `3f3286b8d`).
`premises`, `pattern-fit` and `scenarios` are dropped to keep the fork count near today's six.
It applies to every Discussion-Write that starts after the commit; measure the next three to five loom runs as above, then decide whether to keep it and whether Plan, Webster and Merge get the same treatment.
