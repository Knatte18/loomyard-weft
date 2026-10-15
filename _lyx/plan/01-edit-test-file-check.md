# Card 1 — edit-test-file-check

**Create:**
- `plan:internal/planparser#gateEditTestFile` -> `const gateEditTestFile refGate = "edit-test-file"`
- `plan:internal/planparser#CheckEditTestFile` -> `func CheckEditTestFile(plan *Plan, done []Card) []ValidationError`
- `plan:internal/planparser#ignoredByGoTool` -> `func ignoredByGoTool(segment string) bool`
- `plan:internal/planparser_test#TestCheckEditTestFile` -> `func TestCheckEditTestFile(t *testing.T)`

**Edit:**
- `internal/planparser#ledger`
- `internal/planparser/doc.go`
- `internal/planglyph#index.ValidateFormat`
- `internal/planglyph#index.ValidateRework`
- `internal/planglyph/doc.go`
- `internal/loomcli#loomCLI.validatePlanCmd`
- `internal/loomcli#TestGateParity_PlanGate`
- `internal/loomcli#TestGateParity_ReworkPlanGate`
- `internal/loomcli#reworkParityFixture` -> `func reworkParityFixture(t *testing.T, anchorPath, worktreeRoot string, cardNumber int, uses, edit, verify string, committed bool) *loomCLI`
- `internal/loomcli#TestValidatePlanCmd_ReworkReportsGateFindingsForFirstCardMismatch`
- `contracts/specs/loom-plan-spec.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Rename:**
- `internal/planglyph#TestIndex_AppendsFabricVocabularyName` -> `plan:internal/planglyph#TestIndex_AppendsPlanChecks`

**Intent:** Add the blocking plan check `edit-test-file` (D3, D4, D5) and wire it into the plan index's format and rework validation, never dispatch.

`internal/planparser`, new file `edittestfile.go`:
- `CheckEditTestFile(plan, done)` returns nil when `planLanguage(plan)` reports not-ok, so a `language: none` plan is never checked (D3's bound: such a plan forgoes every glyph check, and has neither a self glyph to read nor D5's member-glyph and handle way forward).
- Otherwise, for every card whose `cardID` is not the `cardID` of a card in `done`, and for every ref of every `TargetGroups` entry whose `Type` is `CardTypeEdit` (never `Card.Targets`, so `Create`, `Delete`, `Rename`, `Move`, `Prosa`, `Custom` groups and `Uses` are never read), it asks `lookup(gateEditTestFile, ref)` and skips any ref whose disposition is not `dispKeep`.
  A kept ref is parsed with `parseGlyph`; a parse error (which `glyph-malformed` already reports) or a member glyph (`!IsSelf()`) is skipped, and the path comes from `Glyph.UnitPath()` alone (`PATTERN-glyph-conversion-chokepoint`).
  No branch reads `UnitPath`'s ok result: past the `language` gate the glyph is a Go glyph, for which quarry's `Glyph.UnitPath` always answers ok, so a not-ok skip would be unreachable.
- A path that does not end in `_test.go` (`strings.HasSuffix`, so `foo_test.go.orig` passes) is skipped, and so is a path any of whose `/`-separated segments, the file name included, `ignoredByGoTool` reports: `testdata`, `vendor`, or a name starting with `.` or `_`.
- Each remaining ref is one finding `{Check: "edit-test-file", Card: cardID(card), Ref: ref}` whose `Detail` names the card and the file, then D5's way forward: name each existing test function, fake or helper the card changes as a member glyph under `**Edit:**`; declare each new one as a `**Create:**` `plan:` handle on the package directory, or on `<dir>_test` for an external test package; a new test file stays a `**Create:**` path; a change confined to the file's header, such as its build constraint, names every member of that file.
- It reads plan text only, so it fires whether or not the file exists, and only refuses.
- Its doc comment says it sits outside `ValidateFormat` and `Validate` because the plan index appends it, and why it takes the done set where `CheckCardFabricReference` filters nothing.

`shape.go`: `gateEditTestFile` joins the `refGate` const block with a comment naming `CheckEditTestFile`'s policy, and `ledger` gains its entry: `refKindGlyph` `dispKeep`, and `refKindPath`, `refKindSymbol`, `refKindHandle` `dispSkip`.
A plain `x_test.go` path a card file writes is a glyph by the time the check runs, since `ParsePlan` canonicalizes every extension-carrying path into its file self glyph; a path-shaped ref left after parsing has no extension, so it is never a test file.
`TestRefGateConstantsMatchLedger` and `TestLedgerCompleteness` then cover the new gate unchanged.

`internal/planglyph`: `index.ValidateFormat` appends `convertAll(planparser.CheckEditTestFile(plan, done))` and `index.ValidateRework` appends `convertAll(planparser.CheckEditTestFile(plan, nil))`: a rework generation is a fresh plan whose prior cards are archived (`PATTERN-plan-generation`), so no card is skipped there.
`index.ValidateDispatch` is unchanged, so a run past its plan gate never halts on the check; the bound is that a plan an earlier binary admitted can now be refused, only at a plan gate whose writer can fix it.
Both methods' doc comments name `edit-test-file`; `internal/planglyph/doc.go`'s sentence on the format and rework validation's appended checks names it beside `card-fabric-reference`.

`internal/planparser/doc.go`: the "Checks outside both entry points" section names `CheckEditTestFile` as one the plan index appends at format and rework validation, taking the told done set.
The sentence that says the glyph->path call is `Glyph.UnitPath, used only by validate.go's disk-existence checks` drops that limit, which is already false, and names `Glyph.UnitPath` as the sole glyph->path call without listing its readers.

`contracts/specs/loom-plan-spec.md`, in the same commit as the check:
- A new validation row `edit-test-file` after `verify-package-missing`: a file self glyph in a card's `**Edit:**` group whose path ends in `_test.go`, one blocking finding attributed to the card with `Ref` the entry; skipped under `language: none`; a path with a segment the go tool ignores is exempt; read from plan text alone; appended by the plan index to format and rework validation, never dispatch, with a done card skipped at format validation and no skip at rework; D5's way forward; and D3's bound.
- The validation-check preamble names it as the fifth check outside both entry points, beside `fabric-vocabulary-name`, which the plan index appends to its format and rework validation.
- The card path resolution sentence that reads ``Glyph.UnitPath` (glyph -> path, used only by the two disk-existence checks below on a **self** glyph; a member glyph is skipped by both, …)`` names `Glyph.UnitPath` as the sole glyph->path call without listing its readers, and states the member-glyph skip of the two disk-existence checks by name (`card-path-malformed` and `path-missing`) rather than of every reader.
- The "Card fields and order" grammar example lists a member glyph such as `` `internal/boardcli#TestList` `` under `**Edit:**` in place of `` `list_test.go` ``; the `**Create:**` `list_json_test.go` example stays, since a new test file is a `Create` path.

`internal/loomcli#loomCLI.validatePlanCmd`'s long help, which names the checks the default and `--rework` modes include beside `--require-approved`'s, names `edit-test-file` too.

Tests:
- `TestCheckEditTestFile` (`internal/planparser_test`, beside `TestCheckFabricVocabularyName`, built from `validCard` with `TargetGroups` set) is one table whose rows are: a `_test.go` file self glyph under `Edit` (one finding, compared exactly as a whole `ValidationError`, `Detail` included, so every clause of D5's way forward is pinned); the same glyph under `Create`, under `Delete` and in `Uses` (none); a test member glyph such as `internal/x#TestFoo` under `Edit` (none); a path-shaped `internal/x/foo_test.go` with no `#` under `Edit` (none, the ledger's path skip, which holds because `ParsePlan` canonicalizes an extension-carrying path into its self glyph first) and a `plan:internal/x#TestNew` handle under `Edit` (none, the handle skip); a glyph-shaped `Edit` ref that fails `glyph.Parse` (none, the parse-error skip); `_test.go` paths under `testdata/`, `vendor/`, a `.`-prefixed and a `_`-prefixed directory segment, and `internal/x/_foo_test.go#` whose base name alone starts with `_` (none each); `internal/x/foo_test.go.orig#` (none); a production `.go` and a `.md` file self glyph (none each, both named by the record's acceptance criteria); the offending card under `Language: "none"` (none); and two offending cards with only the first in `done` (one finding, on the second).
- `TestIndex_AppendsPlanChecks` (renamed from `TestIndex_AppendsFabricVocabularyName`) gains a `check` field and a `language` field per row, counts findings of that check, and sets `plan.Language` from the row, since `minimalPlan` builds a `language: none` plan on which `CheckEditTestFile` reports nothing.
  Its existing rows keep `fabric-vocabulary-name` under `none`; new `edit-test-file` rows under `go`, over a card whose `Edit` group holds `internal/x/x_test.go#`, pin: format with no done card reports one, rework told the plan's first card reports one, dispatch with the card forthcoming reports none, and format with the card passed as done reports none, which pins that `index.ValidateFormat` forwards its `done` set rather than `nil`.
  Its doc comment names both checks.
- `TestGateParity_PlanGate` gains a row `EditTestFile` whose build writes `sub/a_test.go` (`package sub` with an empty `TestA(t *testing.T)` importing `testing`) into the worktree and a `language: go` one-card plan through `plankit.Write` with an `Edit` group targeting `sub/a_test.go` and an `ImpactSummary`; `wantGate` and `wantCLI` are stuck and `wantFinding` is `edit-test-file`, so the check is pinned from a card file's plain path through `ParsePlan`'s canonicalization to both the plan gate and `lyx loom validate-plan`.
  The test's doc comment, which lists its fixture set case by case, names the new row.
- `TestGateParity_ReworkPlanGate` gains an `edit` field per row and a row `EditTestFile` with `edit` `sub/a_test.go`, the same `sub/a_test.go` written into the worktree, card number 2 and committed, whose verdict is stuck and `wantFinding` `edit-test-file`, so the rework plan gate and `lyx loom validate-plan --rework` both report it.
  `reworkParityFixture` takes the `edit` target and, when it is set, adds an `Edit` group holding it beside the card's `Create` group; its other caller, `TestValidatePlanCmd_ReworkReportsGateFindingsForFirstCardMismatch`, passes `""`.
- `cmd/lyx/testdata/test-budget.yaml`'s `internal/planparser` row rises by one for `TestCheckEditTestFile`.

Existing fixtures that list a `_test.go` file under `**Edit:**` and run through the index's format validation: `TestGlyphChain_CallerCoverage` lists `callees/callees_external_test.go` under `Edit` and goes through a slotted index's `ValidateFormat` in one row, but it filters findings by `Check`, so it stays green; no other fixture in `internal/planglyph`, `internal/loomshed` or `internal/loomcli` lists one.

**ImpactSummary:** A new blocking finding at both plan gates and `lyx loom validate-plan`'s default and `--rework` modes, never at webster dispatch; callers of the two index methods (`internal/loomshed/gates.go`) are unchanged.

**Commit:** `1: Refuse a plain test-file path under Edit at the plan gates`
