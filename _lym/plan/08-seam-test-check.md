# Card 8 — seam-test-check

**Create:**
- `plan:internal/loomshed#SeamTestReader` -> `func SeamTestReader(worktreeRoot string) func() ([]impactset.SeamTest, error)`
- `plan:internal/loomshed#seamTestFindings` -> `func seamTestFindings(plan *planparser.Plan, cards []planparser.Card, worktreeRoot string, index planindex.Index, seams []impactset.SeamTest) ([]planindex.Finding, error)`
- `plan:internal/shedrecipe#noFindingsPlanIndex.ResolveGlyphs` -> `func (noFindingsPlanIndex) ResolveGlyphs(worktreeRoot string, glyphs []string) ([]planindex.GlyphAnswer, error)`

**Edit:**
- `internal/loomshed#PlanContext`
- `internal/loomshed#planContextFindings` -> `func planContextFindings(plan *planparser.Plan, cards []planparser.Card, worktreeRoot string, index planindex.Index, pc PlanContext) ([]planindex.Finding, error)`
- `internal/loomshed#ValidatePlan`
- `internal/loomshed#ValidateReworkPlan`
- `internal/loomshed#loomshedAllowedImports`
- `plan:internal/loomshed#TestValidatePlan_CardChecks`
- `internal/shedrecipe#planContextOf`
- `internal/shedrecipe#TestPlanWriteEntry_GateConfig`
- `internal/loomcli#loomCLI.validatePlanCmd`
- `internal/loomcli#TestGateParity_PlanGate`
- `internal/loomcli#TestValidatePlanCmd`
- `internal/loomcli#TestValidatePlanCmd_Overlay`
- `internal/loomcli#TestValidatePlanCmd_ReworkReportsFindingsForFirstCardMismatch`

**Extends:**
- `contracts/specs/loom-plan-spec.md`
- `internal/loomshed/doc.go`

**Uses:**
- `plan:internal/loomshed#goPackageDirs`
- `plan:internal/impactset#FindSeamTests`
- `plan:internal/planindex#GlyphAnswer`
- `internal/planparser#GoPackageCounterpart`
- `internal/planparser#ParseRef`
- `internal/planparser#PathEntryPath`
- `internal/testkit/plankit#Write`

**Intent:**
D6: a card that touches a struct a test reflects over covers that seam test.

`PlanContext` gains `SeamTests func() ([]impactset.SeamTest, error)`; nil skips the check.
`SeamTestReader(worktreeRoot)` returns the reader both builders set: called at check time, it runs `impactset.FindSeamTests` over `goPackageDirs(worktreeRoot)` and flattens the answer in directory order; its error, a `*GuardScanError` included, is returned.
`planContextOf` sets it over `env.WorktreeRoot` and `validatePlanCmd` over `c.env.WorktreeRoot`, beside the tier tags of card 7.
`loomshedAllowedImports` gains `internal/impactset`, with a comment that it parses the markers of told directories and resolves no geometry.

`planContextFindings` gains the index its callers already hold, re-signed above; `ValidatePlan` and `ValidateReworkPlan` pass theirs.
When `plan.GoToolchain()` holds and `pc.SeamTests` is set, it calls the reader once and appends `seamTestFindings` after the tier findings; a reader error is returned.
That error is an infrastructure fault the plan writer cannot fix, since it sits in a test file: it names the file and line and card 5's way forward, which a session that may edit tests takes before the gate is re-run.

`seamTestFindings` resolves every marked struct glyph in one `index.ResolveGlyphs` call (made only when the plan has cards and some marker exists; its error is returned), giving each struct's declaring files.
A card is held to a marked struct `u#T` when a ref of its `Create`, `Edit`, `Extends`, `Delete` or `Rename` groups, with a `plan:` prefix stripped, is `u#T`, a member `u#T.<x>`, the file self glyph of a file declaring `T`, or the package self glyph `u#`.
Such a card covers a test marked for `u#T` when its own target groups name the test's glyph `<Unit>#<Name>` or its `planparser.GoPackageCounterpart`, or its `Uses`, `Delete`, `Rename` or `Move` refs name the test's file, or any of its refs is the package self glyph of the test's unit or of its counterpart unit; or when a later card names the test glyph or its counterpart under `Edit` or `Extends`.
Otherwise it is one blocking `seam-test-unlisted` finding per card, struct and test, with "way forward: list it under Edit by its member glyph when the card changes it, under Uses when it stays as is and no later card targets it, leave it to the later card that edits it, else list it under Edit".
The closing "else list it under Edit" covers a test a later card deletes, renames or moves, where `uses-later-target` refuses the Uses listing and no later Edit or Extends covers it.
Bound, stated in the spec: a method change on a marked struct also requires the listing; a field added through a target this rule does not name is not seen; a test listed under Uses, or edited by a later card for another reason, passes while the card breaks it, and the card's gate test run and review catch that.

Tests:
`TestValidatePlan_CardChecks` gains D6 rows over a temp worktree holding `internal/s/s.go` declaring `type Deps struct{}`, `internal/s/doc.go`, and `internal/c/c_test.go` declaring `TestDeps` marked `//lym:seam internal/s#Deps`, with `SeamTests` set to `SeamTestReader` over it.
A card editing `internal/s#Deps` without the test fails with `seam-test-unlisted` naming card, struct, test and the way forward; so do a card editing the member `internal/s#Deps.Field`, a card creating the handle `plan:internal/s#Deps.Run` (a method of the struct), a card editing the file `internal/s/s.go`, a card editing the package `internal/s#`, a card extending `internal/s#Deps`, a card deleting `internal/s#Deps.Field` and a card renaming `internal/s#Deps.Field` to `plan:internal/s#Deps.Other`;
a card listing `internal/c#TestDeps` under Uses with no later card targeting it passes; a card that also edits `internal/c#TestDeps` passes; a card whose later card edits `internal/c#TestDeps` passes, and so does one whose later card extends it; a card listing `internal/c_test#TestDeps` while the test sits in package `c` passes; a card listing the test's file `internal/c/c_test.go` under Uses passes; a card listing the package self glyph `internal/c#` under Uses passes, and so does one listing `internal/c_test#`; a done card, a `language: csharp` plan and a nil `SeamTests` pass; a marker the parser rejects makes `ValidatePlan` return the scan error; and `ValidateDispatch` over the failing row's plan reports no `seam-test-unlisted`.
`TestGateParity_PlanGate` gains a case whose fixture marks a test for a struct a card edits without listing it: the gate told `SeamTestReader` over the fixture root, the lens-review gate, and the verb's no-flag and `--lens-review` modes all report `seam-test-unlisted`.
Over the tiered fixture card 7 gives them, `TestValidatePlanCmd`'s pair of rows, `TestValidatePlanCmd_Overlay`'s row and `TestValidatePlanCmd_ReworkReportsFindingsForFirstCardMismatch`'s assertion each gain a marked test and a card editing its struct: the no-flag, `--overlay` and `--rework` modes report `seam-test-unlisted` and `--require-approved` does not.
`TestPlanWriteEntry_GateConfig` gains a row: the gate built through `planContextOf` over such a worktree reports it.
`noFindingsPlanIndex`, the index that test's rows run on, gains `ResolveGlyphs`, answering every glyph `GlyphFound` with no files, so the row is held through the struct's own glyph and never calls the nil embedded index.

Docs: `contracts/specs/loom-plan-spec.md` gains the `seam-test-unlisted` row with its holding and covering rules, way forward and bound, and its card rules say a card touching a struct a `//lym:seam` test marks lists the test; `internal/loomshed/doc.go` lists the check and `SeamTestReader`; `validatePlanCmd`'s Long help names it.

**ImpactSummary:** `planContextFindings` gains an index parameter; its two callers sit in the same file and are listed; the new refusal fires only on a go plan with a card touching a marked struct.

**Commit:** `8: Refuse a card that touches a marked struct without its seam test`
