# Clean-room rewrite of the most patched modules, landing first

`clean-room-module-rewrite` · Note · enhancement, landing

## What was seen

Much of lym's code, and its tests, grew by fixing bugs on bugs.
Fix commits over the past month cluster in a few packages:
`loomcli`, `fabricengine`, `landingshed`, `shuttleengine`, then `battencli`, `websterengine` and `driftengine`.
The operator suspects these need a hard rewrite, tests included.

## Already built (shrink estimate, `dcc5e7ac`, 2026-11-18)

The report `docs/research/rewrite/shrink-estimate-2026-11-18/report.md` measured all seven modules and estimated a clean size for `landingshed`.
Duplicate code is small (0.7% of production lines, 1.5% of test lines); the size is in comments (27.4% of production lines) and tests (1.94 test code lines per production code line).
`landingshed` clean is about 4,400 code lines against 5,418, and about 4,000 to 6,000 test lines against 12,585.
Extrapolated to all seven modules, 38% to 55% fewer lines, of which tests are 74% to 80%; the extrapolation is a labelled guess.
`landingshed-spec.md` in the same directory is a drafted spec for `landingshed`: eleven invariants and a rule table, each rule tied to its commit.
It is written as step 1 of the pilot, but cites commit subjects and bodies, not diffs, so step 1 below starts from it and checks it rather than from nothing.

## Idea

Rewrite one module at a time, clean-room, with its exported interface held fixed as the contract.
Interface changes between modules wait for a later round, once both sides are rewritten.

1. Spec: a darn docs run writes what the module must do and why:
   its job, the invariants that always hold, and each special case with its reason.
   It draws the reasons from the module's fix-commit history, never describing how the current code does it.
2. Tests from the spec alone: another run writes new tests without seeing the old code or the old tests.
   The tests use only the module's interface.
   They run against the old code; each failure is either a spec error or an old bug,
   and each is decided by the operator and written into the spec.
3. Code from the spec and the new tests: a darn solid run deletes the old package and its tests in the worktree first,
   then writes the module anew.
   It must pass the new tests and every other module's tests, which this round leaves unchanged.
4. Compare: lines of code and of tests, coverage, gate time, and any behaviour that changed.

## Pilot

Run the pilot on `landingshed` (mid-sized, clear boundary).
It measures one module's cost in tokens, wall time and operator decisions in step 2.
On its result the operator decides whether to go on, and with which of the most patched modules;
the shrink estimate's mechanical counts may set the order.
Modules with no shared code can then be rewritten in parallel.
