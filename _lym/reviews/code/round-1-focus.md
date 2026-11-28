---
round: 1
focus:
  - "Root agreement: does `reviewTargetSnapshot` in `internal/shedrecipe/entries_burler.go` root overlay paths where reviewers and fixer read them, for every geometry that builds a recipe Env (compare `internal/hubgeom`, `internal/standalonegeom`, `internal/shuttleparity`), and does any other reader of the snapshot or of a round's target paths still use a different root?"
  - "Empty-target error: follow each caller of `TargetSnapshot.ID`, `Take` and `ChangedSince` when no target file exists, including a fixer that removes every target file, a target that appears or vanishes between rounds, and a replayed round. Does each outcome meet the board entry's expectation that empty input fails loudly and never reads as unchanged?"
  - "Test coverage: do the changed tests fail on the old code for both behaviours, and is any behaviour the diff adds left uncovered, such as a round over a missing target recording no target id?"
  - "Change description: check each claim in `_lym/landing/summary.md` against the code, and check whether its `## Merge danger` door and blast radius match what the diff can affect."
---

The diff is small and has two parts: the snapshot root moves to the anchor path, and an overlay id over no file becomes an error.
The questions above are about the callers and geometries the diff does not touch.
