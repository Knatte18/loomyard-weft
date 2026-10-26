---
verdict: APPROVED
---

The diff makes the change the amended board entry asks for.
`CheckCardFabricReference` runs `matcher.MatchPath` over each card id, so every id the audit's bare-word pattern would flag is refused at the plan gate, and an id holding the suffix mid-name passes.
The finding names the card and the matched id and gives the rename as its way forward.
It keeps the `FabricReferenceMatcher` seam, adds no import to `planparser`, and reuses the `card-fabric-reference` check ID.
`internal/fabricengine/refscanner.go` is untouched.
The spec entry, the package doc and the function's doc comment describe the new hit.
Both new test rows sit in the existing tier0 table, derive the suffix, and duplicate no existing row.
Existing rows use slug `alpha`, so their count, order and `Card` field do not change.
The commit declares no PATTERN change and carries no `Pattern-Departure:` trailer.

## Rejected

- The done-card exemption (lens `code-affected-set`, LOW, scope): the new finding ignores the done set that `edit-test-file` honors, so format validation also refuses a landed card with a suffix-ending slug. The decision record's Open risks names this behavior and accepts it, so the diff matches the answer key.
