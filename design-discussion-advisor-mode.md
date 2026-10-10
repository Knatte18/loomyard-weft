# Config switch: every advisor gets every question, or each advisor owns an area

`discussion-advisor-mode` · Note · enhancement, loom, stencil, config

## Seen

The chair-and-advisors Discussion (`discussion-multi-llm`) sends every question batch to every advisor, by the operator's PR-Gate ruling on PR #504 (2026-10-10).
The first draft instead gave each advisor its own area of the design, which narrows each advisor's view but spreads the study.

## Idea

Support both, selected per hub in `loom.yaml` (for example `discussion_advisor_mode: all | areas`, default `all`), with the value passed to the chair stencil as a marker.
Decide after `discussion-multi-llm-bench`, which can compare the two modes on the same tasks.
