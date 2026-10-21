# Round 1 fixer-report

## Changes

- F1 (LOW): fixed as proposed, commit `5777f414`.
  `TestFabric_ReadTaskPatch` gained the subtest `RootScratchStaysOutOfThePatch` in `internal/fabricengine/taskpatch_integration_test.go`, backed by the helper `readPatchAroundScratchWrite`.
  It wires the fixture warp's root `.scratch` as a link to a directory outside the warp (`fslink.CreateDirLink` plus a `/.scratch` exclude line), writes a file under it, and asserts the `TaskPatch` is unchanged.
  It builds a second fixture with a real `.scratch` directory and a `/.scratch/` exclude line, and asserts both fixtures give the same `FullHash` and `PatchID`.
- Self-check follow-ups, same file, one commit after the F1 commit:
  the linked fixture's `Full` is asserted free of `.scratch`, the note name uses `scratchNoteName`, and the test's doc comment names the new behavior.

## Self-check

- fix-consistency, LOW: `TestFabric_ReadTaskPatch` doc comment omits the new behavior — fixed as proposed.
- fix-consistency, LOW: `"note.txt"` literal repeats `scratchNoteName` — fixed as proposed.
- fix-proposal, LOW: the before/after check on the linked fixture cannot fail if the exclude is lost — fixed as proposed (direct `.scratch` absence assertion on `Full`).
- fix-proposal, LOW: the test writes its own exclude line instead of the one the production scratch step writes — fixed differently: no change.
  The review's `fix` key names writing the exclude line directly as an allowed route, and driving the real step needs a wired hub, which this fixture does not build.
  `scratchlink_integration_test.go` already pins the line the step writes.
- fix-code-correctness: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
