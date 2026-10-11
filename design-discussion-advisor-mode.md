# Discussion advisors: operator-named areas per advisor on top of the landed all-advisors default

`discussion-advisor-mode` · Note · enhancement, loom, stencil, config

## Seen

The chair-and-advisors Discussion (`discussion-multi-llm`) sends every question batch to every advisor, by the operator's PR-Gate ruling on PR #504 (2026-10-10).
The first draft instead gave each advisor its own area of the design, which narrows each advisor's view but spreads the study.

## Idea

Support both, selected per hub in `loom.yaml` (for example `discussion_advisor_mode: all | areas`, default `all`), with the value passed to the chair stencil as a marker.
Decide after `discussion-multi-llm-bench`, which can compare the two modes on the same tasks.

## Operator-named areas

The operator also wants to choose which area each advisor studies (2026-10-10), rather than leaving the split to the chair.
One shape: each `discussion_advisors` entry carries a model-spec plus an optional area, for example `{model: sonnet[high], area: error-handling}`, and the area names a lens text the advisor stencil includes, the way Burler's review fans name lenses (`correctness`, `error-handling`, `test-gaps`).
An entry without an area answers every question; an entry with one still sees every batch but answers from its lens.
This could replace the `all | areas` switch: the per-entry area is the selection.

## When (operator, 2026-10-11)

Advisors with different roles is a later enhancement.
The first DualDW bench runs one advisor at most, answering every batch as a plain assistant.
