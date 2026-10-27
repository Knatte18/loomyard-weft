# Fixer-report, code round 1

## Changes

All three findings fixed as proposed, one commit each; every change is a comment edit in `internal/boardengine`.

- F1 (`eb6d0254`): `render.go:4` and `render.go:238` now read "dependency layers, Independent and Draft".
- F2 (`558efa81`): the `standingCell` doc comment at `render.go:442` now lists "Ready, Note, Project, Draft, or the name of its dependency layer".
- F3 (`ddac46f2`): `doc.go:176` now reads "Ready holds the open, released tasks with no open dependency", and `layer.go:29` reads "An open task that is not a draft and has no open dependency is readyLayer".

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

Nothing disputed.

## Deferred

Nothing deferred.
