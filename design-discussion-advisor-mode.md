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

## Co-chair (operator, 2026-10-11)

An alternative to an advisor: one seat is a co-chair that does its own exploration of the code and the task rather than only answering the chair's batches, so the draft gets a second independent study.
How the co-chair's findings reach the chair, and who writes the decision record, is open.

## Lens advisors as forks of the chair (2026-10-11)

The operator compares lens advisors with Burler's review fan, whose lens forks each review from one focus.
Burler's lenses are forks of the reviewer: they inherit its context and cached prefix and run on its model.
Seats advisors are fresh sessions on their own models, and each re-reads the code; in the first bench run (`dbench-ohsh`) the advisor's cache reads exceeded the chair's.
A variant: fork the lens advisors from the chair once it has read the task and code, so each lens starts with the chair's study at little cost, at the price of running on the chair's model.
Fresh-session advisors give breadth through different models, chair forks through different focus; a later bench can compare the two.
Lenses need a producer change either way: today the engine and chair stencil send every batch to every advisor with no focus.

## Board advisor in place of parent review (2026-10-11)

The operator asks whether an advisor with a board lens could take over Discussion's parent review, whose job is mostly checking the draft against everything else planned on the board.
Parent review also checks the draft against operator decisions made in the orch chat but not yet on the board, and against runs in flight; the one parent-review reject on 2026-10-11 (`per-role-models` round 1, seed model) was of that kind, which no advisor could have caught.
Proposal: put a board-lens advisor in the second bench round and compare what it catches with what parent review catches before making parent review optional.

## Advisor answers (operator, 2026-10-11)

Advisors write too much.
When an advisor agrees with the chair's pick, it says so in one line with no reasoning, since "I agree because ..." is clutter for the chair.
Every extension, correction or departure is explained.

## When (operator, 2026-10-11)

Advisors with different roles is a later enhancement.
The first DualDW bench runs one advisor at most, answering every batch as a plain assistant.
