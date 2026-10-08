---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: internal/planglyph/planglyph.go:63
    summary: Edited Go comment sentences in planglyph.go and four planparser files still break mid-phrase, the same defect round 1 F1 raised and fixed only for the planglyph locations it named
  - id: F2
    severity: NIT
    class: consistency
    location: internal/planglyph/deleteorder.go:81
    summary: editTarget doc comment is ungrammatical and does not say what the type holds
---

### [BLOCKING] Edited comment sentences still break mid-phrase

**Location:** `internal/planglyph/planglyph.go:63-64`, `internal/planparser/handle.go:3-4`, `internal/planparser/handle_test.go:1-2`, `internal/planparser/shape.go:103-104`, `internal/planparser/validate.go:465-475`
**Issue:** `PATTERN-comment-line-breaks` allows a break inside a sentence only after a semicolon or at a comma before a coordinating conjunction whose clause has its own subject and verb.
Round 1 F1 applied that rule to sentences the diff edited and whose old column-wrapped tail it left; the fixer rewrapped only the locations F1 named, all in `internal/planglyph` and `internal/websterengine`.
These sentences are edited by the same commits and still break mid-phrase, and `lint-comments` passes because it flags only a break the diff creates:
- `planglyph.go:63-64`, the `ValidateDispatch` doc: card 8 rewrote the tail to "gets ValidateFormat's answer less planGatePass's findings, a pass that runs at the plan gates only", and the sentence still breaks between "a caller with none" and "gets".
- `planparser/handle.go:3-4`, the file header: card 6 rewrote "declared/referenced predicates" to "claim/referenced predicates" and joined the tail onto one long line, and the sentence still breaks between "handle-specific parse" and "helpers".
- `planparser/handle_test.go:1-2`, the file header: card 6 rewrote the predicate clause, and the sentence still breaks between "card 9" and "adds".
- `planparser/shape.go:103-104`, the `ledger` doc: card 5 dropped "thirteen" from "covering exactly the thirteen policies", and the sentence still breaks between "per" and "refGate".
- `planparser/validate.go:465-475`, the `checkGlyphMalformed` doc: card 5 removed the "containment-unit-overlap skips it the same way" clause, and the rest of the sentence keeps its fixed-column breaks, for example "collectGlyphTargets / silently drops".
**Fix:** Put each of these edited sentences on one line, or break it only at a semicolon or a comma before an independent clause, and leave untouched neighbouring sentences alone.

### [NIT] editTarget's doc comment does not parse

**Location:** `internal/planglyph/deleteorder.go:81`
**Issue:** The comment reads "editTarget is one member glyph or file an Edit group of card targets."
The sentence has no verb linking "an Edit group of card" to the target, so a reader cannot tell whether the type holds a target, a group or a card.
The struct holds one Edit-group ref of one card, together with its shape and disk path.
**Fix:** State what the type is, for example "editTarget is one Edit-group ref of a card that is a member glyph or a file, with its shape and disk path."

## Range and checks run

The branch has no rework round (`_lyx/loom/rework/` is absent), so the range is `git diff $(git merge-base main HEAD)..HEAD`, base `fd5b0f58e`.
Build, `go vet` under the untagged, `integration`, `tmux` and `llm` tiers, and the full `go test ./...` pass.
`go test -tags integration` passes for `internal/planglyph`, `internal/planparser` and `internal/websterengine`.
`lyx loom lint-comments --base fd5b0f58e` reports ok.
The `TestTestBudget` budget test passes with the raised `internal/planglyph` row.
Every card's targets are present in the diff, and the spec rows match the order the checks run in the code.
Card 5 is the only card with a `Delete` group; its four files are gone and nothing references them, so assert-no-callers holds for each.
No card carries a `Rename` group.

## Round 1 regression check

Read after this review was saved.
Round 1 F2 to F9 hold: every `redundancy.go` key is a `UnitPath` result (F2), the spec sentences sit on their own lines (F3), repeated targets give one finding with a test row each (F4), `isInterfaceMethod` tests `func (` and `func(` with a func-named regression test (F5), the skipped-directory row exists (F6), the `doc.go` overview names `planGatePass` (F7), the unreachable row is gone (F8), and the repeated plan-gate assertion is gone (F9).
Round 1 F1 holds for the locations it named and is incomplete for the ones in the finding above.

## Focus answers

- **Comment line breaks.**
  Yes: five edited sentences still break mid-phrase, listed in F1; no break the diff itself creates is mid-phrase.
- **Glyph chokepoint in the redundancy check.**
  Every key in `redundancyOwnerOf`, `renameToSideUnits` and the `containers` map comes from `Glyph.UnitPath` or `path.Dir` of its result; `redundancy.go` never reads `Glyph.Unit`.
- **Repeated targets.**
  `planGatePass` dedupes a card's targets with a per-card `seen` set, and `coverageSubjects` dedupes by card and ref, so one finding per file and member and one per subject and file result.
  Rows: `TestGlyphChain_RedundantFile` "file beside a member listed twice", `TestGlyphChain_CallerCoverage` "delete of a member listed twice", and `TestValidate_RedundantPackageTarget` "each contained target is its own finding".
- **Interface-method detection.**
  `isInterfaceMethod` tells a declared method by the `func (` or `func(` prefix, so `funcLike() int` is an interface method, and `renameDeclSource` prepends `func (Owner) ` for it.
  `TestGlyphChain_FuncNamedInterfaceMethod` covers the arrow and the rename.
- **Markdown line breaks.**
  The spec's rewritten handle-checks paragraph and the split `glyph-malformed` row put each sentence on its own line, and the table cells stay on one line.
  The edited stencil and rubric text follows the same rule.
- **Package overview accuracy.**
  The `internal/planglyph/doc.go` overview says `ValidateFormat` and `Validate` append `planGatePass` after `resolvePass`, and `ValidateDispatch` shares `resolvePass` but never runs it; the code matches.
