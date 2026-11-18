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
