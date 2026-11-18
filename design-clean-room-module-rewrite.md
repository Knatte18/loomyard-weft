# Clean-room rewrite of the most patched modules, landing first

`clean-room-module-rewrite` · Note · enhancement, landing

## What was seen

Much of lym's code, and its tests, grew by fixing bugs on bugs.
Fix commits over the past month cluster in a few packages:
`loomcli`, `fabricengine`, `landingshed`, `shuttleengine`, then `battencli`, `websterengine` and `driftengine`.
The operator suspects these need a hard rewrite, tests included.

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
a study of those packages may set the order.
Modules with no shared code can then be rewritten in parallel.
