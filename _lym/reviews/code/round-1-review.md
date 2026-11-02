---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    origin: lens:code-tests
    location: internal/planparser/join_test.go:64
    summary: No test row pins that an export met only by a Delete target or a re-sign arrow is refused, at the join or for a Delete in the part check
    fix: In internal/planparser/join_test.go add refusal rows to the table that holds the "an export no card delivers" case, one whose part exports a path its only card deletes and one whose part exports a member its only card re-signs with an Edit arrow, each expecting the join refusal "no card creates its export". In internal/loomshed/partcheck_test.go add a TestValidatePart row beside "export a card only re-signs" whose card is a Delete of the exported path and whose wantCheck is part-export-missing. Fold the cases into the existing tables as rows, never as new test functions.
  - id: F2
    severity: LOW
    class: consistency
    origin: lens:code-correctness
    location: internal/planparser/exportname.go:498
    summary: CheckExportName re-implements the handle-malformed rules apart from checkHandleMalformed, under the default language and without the fileUnitRuleApplies gate, so the Parts-Write gate and validate-plan --part can disagree
    fix: In internal/planparser/exportname.go state in the CheckExportName doc comment that the file-unit rule is applied to every handle, since an export is always a Create declaration, and that the language is the plan format's default. Drop no check. If the file-unit finding is to stay in step with checkHandleMalformed, move the shared message and the Go-and-.go-suffix test into one unexported helper in internal/planparser that both validate.go's checkHandleMalformed and CheckExportName call, with checkHandleMalformed still applying fileUnitRuleApplies before it calls the helper.
  - id: F3
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: internal/loomshed/partcheck_test.go:147
    summary: The gate tests pass an empty temp directory as worktreeRoot, so nothing fails if NewPartsGate or validate-parts hands Validate the wrong root
    fix: In internal/loomshed/partcheck_test.go add a TestNewPartsGate subtest that writes a file under the worktreeRoot it passes, lists that path in a part's exports, and expects a failed gate whose findings hold export-existing. In internal/loomcli/validateparts_test.go add the same case as one row of the existing validate-parts table, expecting the export-existing finding.
---

### [MEDIUM] No test pins the Delete and re-sign refusals of ExportMet

**Location:** internal/planparser/join_test.go:64, internal/planparser/join.go:162, internal/loomshed/partcheck_test.go:207
**Issue:** The Goal's second change has the gate refuse an export that names an existing member, and the diff narrows `ExportMet` so that only a Create target or a Rename destination meets an export. The only row that pins the narrowing is the part-check row "export a card only re-signs". The diff turned the join test's Delete and re-sign exports into parts with no export at all, and added no refusal row in their place. Restoring `case CardTypeDelete` in `ExportMet`, or the re-sign branch, would pass every test, and no test reaches the join's own changed refusal text. This is a behavior the diff changes with no covering row.

### [LOW] The Parts-Write export name checks can disagree with validate-plan --part

**Location:** internal/planparser/exportname.go:498
**Issue:** `fabricVocabularyBreak` is shared by `CheckFabricVocabularyName` and `CheckExportName`, so the fabric-vocabulary verdict cannot drift. The `handle-malformed` rules are not shared: `CheckExportName` repeats the no-unit and file-unit tests of `checkHandleMalformed` and shares only the check-id constant, and it judges under the default language with no `fileUnitRuleApplies` gate. For the case the Goal names, a Create declaration of a Go member, both paths agree. They differ for a part whose plan sets another language, and for a handle that a part creates only as a Rename New side. In both, the parts gate can refuse what the part gate accepts, or pass what it refuses. The report states the language limit as a design choice, so the finding asks only that the code say so and that the two copies not drift.

### [LOW] The worktree root passed to the parts check is not pinned

**Location:** internal/loomshed/partcheck_test.go:147
**Issue:** The `export-existing` refusal is pinned at `Validate` through a real file under a temp worktree. `NewPartsGate` and the `validate-parts` verb are the two callers that choose the root, and every test of them hands an empty temp directory or a root with no matching file. A caller that passed the anchor path, or an empty string, as the worktree root would pass every test and refuse nothing.

## Rejected

- Stencil line 38 of `loom-template-parts.md` ("creates, re-signs or deletes") reads against the new Step 4 (lens code-affected-set): it states when a part depends on another, which concerns use and not exports, and it is true under the new rule.
- `loom-plan-checks-spec.md` does not list the parts gate as a new origin of `handle-malformed` and `fabric-vocabulary-name` (lens code-affected-set): `loom-parts-spec.md`'s check table, the spec that owns the parts gate, lists both, and the checks' own entries describe the checks, not their callers.
- Spot exports have no handle-name-check row (lens code-tests, NIT): `checkExports` is one function over both loops, and the part row already pins the name check and its prefix.
- The C# and Rename-sourced cases as two findings (lens code-correctness): kept as one, F2, since they have one cause.
