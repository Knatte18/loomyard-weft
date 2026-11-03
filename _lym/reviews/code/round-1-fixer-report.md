# Burler fixer-report — code, round 1

The gate `lym gate test ./internal/boardengine` passed after the edits: tier0 653 targets discovered, 651 passed, 0 failed, 2 skipped.
Each finding is its own commit.

## Changes

- **F1** (`bd6e11fb`), fixed as proposed in `internal/boardengine/dependency.go`.
  `met` and `standsFor` now call `metBelow` and `standsForBelow` with a fresh path set.
  A project already on the path reads as not met in `met` and adds nothing in `standsFor`, so a project naming itself or two projects naming each other end the walk.
  Rows added: `ProjectsNamingEachOtherAsTheirProject` and `ProjectNamingItself` in `TestDependenciesPending`, and two cycle rows in `TestComputeLayers`.
  `layer.go` is left unchanged, as the "Fixed differently" entry below says.
- **F2** (`efb21fe2`), fixed as proposed.
  A seventh row in `TestRenderReadmeGolden` holds a task depending on an open project, whose After cell is `P1`, and a task depending on a plain project whose only landing member is done, whose After cell is empty and which stands in Ready.
  The row's doc comment line is added.
- **F3** (`41a7ba20`), fixed as proposed.
  A `TestWriteSiblings` row seeds a plain project with one done member and a sibling waiting on the project, and expects `setDeps("X", "P")` to report no siblings.
- **F4** (`0acf0ec9`), fixed as proposed.
  The `render.go` file header now names the After column's entries as those each waits on that are not done, a project judged by its landing members.

## Fixed differently

- **F1, `layer.go` `getDepth`.**
  The finding proposed giving `getDepth`'s project branch the same visited guard.
  I did not add one: `ComputeLayers` runs `detectCycleDFS` first, and it follows the same non-met `waitsOn` edges with a gray/black check.
  A membership loop of unmet projects returns `cycle detected involving <slug>` before `getDepth` runs, so `getDepth` never walks a loop.
  The `met` guard is what lets `detectCycleDFS` reach that error, and the new `TestComputeLayers` rows assert it (`dependency.go` `metBelow`, `layer.go:62-91`).
  The fix-affected-set and fix-proposal forks read the same code and agree.

## Disputed

- **Self-check finding, LOW, from fix-consistency and fix-proposal: the README intro line `render.go:128` still says "An entry waits only on the open entries it names under After".**
  Ground: premise false.
  Evidence: `Dependencies`' type comment in `dependency.go` says "A plain project is done when its status is done, or when it has a landing member and every landing member is done".
  A project the After column drops is therefore done in the sense that decides a dependency, and an entry shown under After is not done, so "open entries" still holds as the line is read.
  The review rejected the same generic wording in the `doc.go` Ready paragraph on that ground.

## Deferred with reason

None.

## Self-check

- fix-code-correctness: no findings.
- fix-affected-set: no findings.
- fix-consistency, LOW, `render.go:128` README intro line: disputed, premise false (see Disputed).
- fix-consistency, LOW, `render.go:137` Ready meaning line "Waits on nothing open": disputed on the same ground; the fork itself calls it generic wording that is arguably still true.
- fix-proposal, LOW, `render.go:128` README intro line: merged with the fix-consistency finding above, disputed.
