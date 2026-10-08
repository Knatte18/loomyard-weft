---
round: 4
focus:
  - "D5 in `_lyx/discussion/decision-record.md` places the Edit-head-to-member match, which calls `quarry.Name`, inside `resolvePass` in `internal/planglyph`. `internal/planglyph/chokepoint_enforcement_test.go` (`TestChokepointCallSites_PinnedToTheirGuardedFunctions`, `chokepointGuardedFuncs`) fails any `quarry.Name` call outside `CanonicalizeHandles`, and `CanonicalizeHandles` feeds every `quarry.Name` result into its canonicalization maps. Settle whether D5 names a seam that passes that test, and whether the head's result is kept out of those maps."
  - "Scope in `_lyx/discussion/decision-record.md` lists the Plan-Review rubric change; D6 and the acceptance criteria define the cards that deliver it. Settle whether Scope, D6 and the acceptance criteria name the same set of rubric findings."
  - "D3 gives a `Rename` to-side handle the unit of its pair's Old glyph rather than the draft handle's spelling, and the first gate runs the pure pairings before `CanonicalizeHandles`. Settle whether the acceptance criteria require a test where the two rules give different answers."
---

Each entry names a site to read and a question to settle; none states what the answer is.
