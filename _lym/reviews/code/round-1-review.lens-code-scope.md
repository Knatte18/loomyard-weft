```yaml
lens: code-scope
outcome: reported
```

Task: the code-scope lens. I traced each change in the diff to the decision it builds, and each decision to the changes that build it.

**Decisions (a decision record alone):**
- **G1**: under the board entry's `## Goal`, the proposal checks apply the board's slug rules, the length limit included, so a slug that is too long is caught before the operator approves.
- **A1**: the `Added after Discussion` entry "No test seam for the red-first overlay: the diff's spec change blocks it". It waives the red-first gate and asks for no code change.

**Per change:**
- **C1**: `internal/boardengine/task.go` exports `ValidateEntrySlug`, and `NewTask` now calls it. `internal/boardengine/doc.go` gains the matching "# Slug rule" lines. This is the shared length and root rule that G1 needs `raddleparser` to receive. It traces to G1.
- **C2**: `internal/raddleparser/check.go` adds `BoardContext.ValidateSlug`, `CheckSlugInvalid` and the `slugs()` branch. This is the check that G1 asks for. It traces to G1.
- **C3**: `internal/loomcli/raddlecontext.go` sets `ValidateSlug` to `ValidateEntrySlug` followed by `fabricengine.ValidatePairSlug`. This wires G1's rules into the context that the gate, `raddle-propose` and `raddle-approve` read (`raddleCheckFor`, raddlecontext.go:136-144). It traces to G1.
  - The pair slug rule goes beyond the length limit. It is still covered: the goal says "the board's slug rules, the length limit included", and boardengine/doc.go:264 documents the pair rule as part of the board's "# Slug rule".
- **C4**: the `slug-invalid` bullet in `contracts/specs/raddle-proposal-spec.md`. This is forced by C2, because the spec lists every check.
- **C5**: the `internal/raddleparser/doc.go` lines. These are forced by C2.
- **C6**: the tests. These are the `TestCheck` rows, the `TestCheck_DetailNamesTheWayForward` table, the `fixture_test.go` `refusedSlug` hook and the `TestRaddleBoardContext` loop. They are forced by C2 and C3.
- **C7**: commit 50e49b8d keeps the context test off the weft naming package, and commit 9c624e9e breaks the comments at sentence boundaries. Both are fixes to C3, C5 and C6, so they are forced.

**Per decision:**
- **G1**: built by C1, C2 and C3, with C4–C7 forced.
- **A1**: this is a waiver of the red-first gate, so no change in the diff needs to build it. The summary's Tests section records the waiver, and that is consistent.

**Findings:** none. Every change traces to G1 or is forced by a change that does, and every decision is built or needs no build.

**Tool calls:** none.
