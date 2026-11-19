# Design: simplify interfaces that make every caller handle too much

`design-simplify-callers` · Note · enhancement, fabric, landing

## What was seen

Reading `landingshed` for the shrink study found design that spreads code over every caller:
a four-value return handled at about 60 call sites where one result type would do,
near-identical helpers with different names and types (about 340 lines, of which a duplicate detector saw 36),
single-caller or test-only seams and adapters, and a lock-release layer around the fixer that a different order would remove.

## Idea

Per module, a run lists such spots with a line estimate each, then simplifies them without changing behaviour,
behind the module's existing tests. Start with the largest modules; `fabricengine` (36,000 production lines, about 1,100 functions) first,
since the operator finds it disproportionately large for its job.

## Built so far

`landingshed` is done as the pilot (`landingshed-design-simplify`, 2026-11-18): one `*ending` return type for every landing step
in place of the four-value return, and merged near-identical helpers shared by Finalize, PR-Review and Parent-Align.
The pilot's purpose was to measure whether its line estimates hold before the same pass is aimed at `fabricengine`;
read that landing's commit message for the measured numbers.
What remains is the other modules, `fabricengine` first.
