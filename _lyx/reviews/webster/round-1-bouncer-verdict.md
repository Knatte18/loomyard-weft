---
verdict: BLOCKING
rationale: "Fix commits for import-allowlist stale detection, the CLI-walk bogus pin, known-finding exemptions, case-folded dropped-concept claims, per-platform tmux isolation, kit import resolution, the step-envelope docs and the sectionOf case all hold, and build, vet and both test tiers pass on HEAD; but the test-accounting decision's before-and-after leaf test counts are still recorded nowhere (the fixer report's section is a PLACEHOLDER), and cmd/lyx/seamsignature_test.go:7-8 and internal/loomrecipe/shape_test.go:1-2 are changed comment lines still wrapped mid-sentence against CONSTRAINTS.md's Comment Line-Break Convention."
---

# Webster-Review round 1 — Bouncer verdict

Range: `_lyx/shed/test-suite-derived-guards/status.json` names `main` as `product.parent`, and `_lyx/loom/rework/` does not exist, so the subject is `git diff $(git merge-base main HEAD)..HEAD`, cards 1–34 plus the fixer's commits `ed620249c` through `7e5678496`.
The review's range determination agrees.
The review carries no `## Focus departures` section, so there is nothing to ratify or reject.

On HEAD, `go build ./...`, `go vet ./...` (untagged, `integration`, `smoke`), `go test ./...` and `go test -tags integration ./...` all pass.

## Ratified and verified as fixed

- **Import-allowlist stale detection (cards 8, 9).**
  `scankit.AssertImportAllowlistNoStale` reports allowed paths no production file imports, and every `seam_enforcement_test.go` in cards 8 and 9 that carries an import allowlist calls it.
  `internal/shuttleengine/seam_enforcement_test.go` is a deny check with no allowlist, so it needs no change.
  Card 7's leaf guards stay on `AssertImportAllowlist`, as card 7 requires.
- **CLI-walk bogus pin (card 11).**
  `bogusProblems` requires `unknown subcommand` in the envelope, and `config#bogus` is now a listed finding with an accurate reason.
- **Known-finding exemptions (card 11).**
  `checkInvocation` runs every assertion for an exempted group, skips with the problems listed, and fails with "now meets the invariant" once they are gone.
  The `board notes#bare` reason now matches the binary.
- **Dropped-concept claims (card 24).**
  `droppedConceptClaims` folds case for `oversized` and `chain` and keeps `## Scope` case-sensitive, matching the merge-base `TestTemplates_NoDroppedBatchConceptsRemain` exactly.
- **Tmux isolation per platform (card 34).**
  The scan checks every (tag set, platform) pair, and `fileNamePlatform` follows go/build's suffix rule, including the bare `linux_test.go` case; fixture cases cover both shapes.
- **Kit import resolution (card 34).**
  `kitImportName` binds the kit by import path, and the unqualified `Main` call counts only under `internal/testkit/tmuxkit`; fixture cases cover alias, impostor and both unqualified sites.
- **Step-envelope test docs** are gone, and **`TestSectionOf`** covers the trailing-clause heading match and the `## Onerous` near miss.
- The fixer's extra commit `4167b6abf`, reattaching the doc comments that card 29's helpers had split from their functions, is a correct fix within the diff.

## Still open

1. **Leaf test counts.**
   The overview's `test-accounting` Shared Decision and the decision record's acceptance criteria require leaf test counts recorded before card 1 and after the last card.
   No commit body, no `_lyx/webster` artifact and no other branch artifact carries them, and the fixer report's section for this item is a literal `PLACEHOLDER`.
   This is the implementation failing the plan's own requirement, not a finding against the plan, and a commit body on the branch satisfies it.
2. **Two changed comments still wrap mid-sentence.**
   `cmd/lyx/seamsignature_test.go:7-8`: both lines are rewritten, and the sentence breaks at "asserts only / Short, bare-group listing", which is not a clause boundary.
   `internal/loomrecipe/shape_test.go:1-2`: both lines are rewritten, and the sentence breaks at "assertions over / New's built list".
   CONSTRAINTS.md's Comment Line-Break Convention governs new and changed comments, so these are in scope.
   Every other added comment line in the diff breaks at a sentence or clause boundary.
