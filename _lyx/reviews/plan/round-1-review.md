---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/plan/05-caller-scan-types.md
    summary: card 5's untagged-tier seam reaches only planglyph's own tests, so webstercli's untagged TestValidateCmd_Envelopes spawns go list through planglyph.Validate, and the card's claim that no other package's untagged test does is false
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/05-caller-scan-types.md
    summary: card 5 sends an objectless identifier in a checked file through the scan, but the re-signed scanReferences takes only a file list, so the per-identifier fallback D2 requires has no mechanism and a whole checked file would be re-scanned by name
  - id: F3
    severity: LOW
    class: design
    location: _lyx/plan/04-caller-scan-import-path.md
    summary: card 4 drops a method match whose selector's left side is an imported name, but its Intent does not state what that lets through (a method call on a local that shadows an import name) or what bounds it
---

### [BLOCKING] Card 5's untagged-tier seam leaves another package's untagged test spawning `go list`

**Location:** `_lyx/plan/05-caller-scan-types.md`, the `**Intent:**` paragraphs on `testmain_test.go` and "Before landing, grep the other packages' untagged tests …".

**Issue:** The decision record's `PATTERN-test-speed` constraint says untagged tests do not spawn and reach the fallback through an injected loader or pure inputs.
Card 5 injects only through `defaultTypesLoader`, an unexported package variable that `internal/planglyph`'s own `TestMain` replaces, and asserts that no other package's untagged test runs a plan gate over a plan that deletes or re-signs a member.
That premise is false.
`internal/webstercli/cli_test.go` carries no build tag, and `TestValidateCmd_Envelopes`'s rows "a delete before a later card's Edit that still references it is refused under the deleting card" and "the same delete after the card that edits the reference passes" write `deleteOrderPlan` (card `deleter` Deletes `sub#Foo`, language defaulting to `go` through `planLanguage`) over `plankit.Repo(t, deleteOrderFiles)`, whose `sub/b.go` calls `Foo`.
The `validate` verb runs `planglyph.Validate` with no run begun (`internal/webstercli/validate.go`'s `scopedValidate`), `Validate` reaches `planGatePass` because `resolvePass` returns findings and no error, and `callerCoverageFindings` gets the subject `Foo` with candidate `sub/b.go`.
After card 5, that load goes through `defaultTypesLoader`, the real `goListLoader`, which webstercli's test binary cannot replace, so the untagged tier spawns `go list` twice per run.
The card's "grep before landing" step would find this only at implementation time, and the plan gives the implementer no way forward that keeps card 5's targets sufficient.

**Fix:** Give card 5 a spawn-free path that covers this test and state it in the Intent: for example, the load is skipped without spawning when the told worktree root holds no `go.mod`, since a root-module load from there cannot succeed and D2's fallback answers it, which keeps `deleteOrderFiles` (no `go.mod`) on the scan.
Name `TestValidateCmd_Envelopes` as the site this protects, and replace the "grep before landing" sentence with that statement.
If the chosen fix instead changes the webstercli test, list `internal/webstercli/cli_test.go` under card 5's `**Edit:**`.

### [MEDIUM] The per-identifier fallback has no mechanism in the re-signed `scanReferences`

**Location:** `_lyx/plan/05-caller-scan-types.md`, "Every candidate file not in `checked`, and an identifier named like a subject that has no object in a checked file, goes through card 4's scan, re-signed `scanReferences` taking the file list." and the arrow `scanReferences(root string, files []string, subjects []coverageSubject)`.

**Issue:** D2 sends to D3 the occurrences "in that file or at that identifier" that lack type information.
Card 5 states both halves, but the scan it names takes whole files and returns every name match in them.
Passing a checked file through it for one objectless identifier re-reports, by name, every typed non-reference in that file: a `u.M()` on another type comes back as part of the collapsed informational finding, and a package-level reference already counted by `typedReferences` is counted twice.
The card does not say how the scan's answer is narrowed to the objectless identifiers' lines, and neither `typedReferences`' result nor `scanReferences`' signature carries those lines.

**Fix:** State in card 5's Intent how an objectless identifier in a checked file reaches the scan: for example, `typedReferences` also returns, per checked file, the lines of identifiers named like a subject that have no `Uses` object, and `callerCoverageFindings` keeps a scan match in a checked file only at those lines.
If that needs a parameter or result change, write it into the `typedReferences` Create head or the `scanReferences` re-sign arrow.

### [LOW] Card 4 does not bound the method matches its import-selector exclusion drops

**Location:** `_lyx/plan/04-caller-scan-import-path.md`, the bullet "a method subject is referenced by `.` and its identifier, except when the identifier left of the `.` is a name the file imports".

**Issue:** The exclusion narrows the name scan, a guard: a method call whose receiver is a local variable or parameter that shadows an imported package's name (`errors.M()` after `errors := …`) is no longer matched, not even informationally.
The Intent names the class it removes (`batcher.Batch`) but not this let-through, nor its bound: after card 5 it applies only to files without type information, where the typed path does not see the call.

**Fix:** Add one sentence to card 4's Intent: the exclusion also drops a method call on a local name that shadows an import, and, once card 5 lands, only in a file the load did not type-check; also state it in the spec's `caller-uncovered` row that card 4 rewrites.

## Focus directives settled

- **`internal/planparser`'s doc where it lists checks:** not a defect.
  The exhaustive check list is `validate.go`'s file comment, which card 3 updates.
  `doc.go`'s sentence lists checks as examples ending "and so on" and says they are implemented by `Validate` in `validate.go`, which stays true since `validate` calls the new check.
- **Test economy, cards 1, 3 and 5:** not a defect.
  `TestNestedModules` covers a new exported function no test reaches; `targetdirs_test.go` holds one test per exported function (`TestCardTargetDirs`, `TestRefFile`) and neither covers it.
  `TestValidate_VerifyNestedModule` follows `validate_test.go`'s one-test-per-check layout, and no existing `TestValidate_*` covers the new check.
  `TestGlyphChain_CallerCoverageTyped` needs the `integration` tier for the real load; `TestGlyphChain_CallerCoverage` is untagged and the existing integration tests (`TestGlyphChain_Delta`, `_UnreferencedHandle`, `_ReworkGenerations`, `_RedundantPackage`) do not cover caller coverage, so no existing test can take its rows.
  The budget rows match: `internal/planparser` and `internal/planglyph` both sit at their rows (82 top-level tests each, `TestMain` excluded), so each new top-level test needs its raise.
- **Card order across cards 4 and 5:** consistent apart from F2.
  Card 4 keeps `scanReferences`, `callerCoverageFindings` and `planGatePass` at their current signatures, so card 5's arrows re-sign from today's heads; card 5's `coverageSubject` field and `isSubjectObject` build on card 4's `importPath`; every caller of a re-signed function is in `internal/planglyph` and on card 5.
  No card uses something a later card creates: `NestedModules` and `ModuleOf` come from card 1, and card 6's pointer at `verify-nested-module` follows card 3's row.
- **Verify coverage:** not a defect.
  Every targeted package (`contracts/stencils`, `internal/loomcli`, `internal/planparser`, `internal/planglyph`, `internal/websterengine`, `cmd/lyx`) is in the one root module, so `go test ./...` and `go test -tags integration ./...` run them; the order matches the template's stated order.
- **Attack surface, cards 2, 3 and 5:** card 3 states that it only refuses and what it misses; card 5 states the interface-call let-through, the assignment-site gap and the degradation to card 4's scan; card 2 adds a stricter step and removes none.
  Card 4's narrowing is F3.
- **Fidelity, D1–D11 and the Constraints:** every Decision is carried (D1, D2 card 5; D3, D4 cards 4 and 5; D5 cards 4 and 5; D6 card 1; D7 card 2; D8 card 3; D9, D10 card 6; D11 card 7), and every Constraint is carried except the `PATTERN-test-speed` clause F1 names.
  The grep-before-landing step is no added scope; F1 covers its false premise.
  The budget-row edits on cards 1, 3 and 5 are not D11 scope: they are the raise `cmd/lyx/testdata/test-budget.yaml`'s own header requires in the commit that adds the tests.
