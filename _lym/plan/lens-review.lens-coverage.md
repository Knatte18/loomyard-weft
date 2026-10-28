```yaml
lens: coverage
outcome: reported
```

Coverage lens: I checked that every decision, acceptance criterion and new branch of the plan has a card and a test row that would fail if it broke.

**BLOCKING**
- **Card 1, D2 darn caller: a row contradicts the card's own Intent.** The Intent says that on a clean `Validate` the darn caller returns the D1/D2 findings with the history findings appended, so the history check runs. D2 says the same: "the history check keeps running only on a record `Validate` finds clean". But the planned `TestNewDiscussionGate` darn row gives a record with a repeated id and otherwise valid sections, and its history seam "fails the test when called". Validate is clean on that record, so the history check runs and the row fails against a correct implementation. Evidence: `_lym/plan/01-decision-id-check.md` Intent ("on a clean `Validate` it returns these with the history findings appended") against its Tests paragraph ("never calls the history seam").
  - Fix: let the history seam run on that row. Add a separate darn row whose record also misses a section, which then gets Validate's findings plus the D1 finding and no history call.
  - That second row is also missing today: no row reaches the "darn caller, Validate found defects, D1/D2 still appended" branch.

**MEDIUM**
- **D4, unpartitioned quarry ambiguity has no row.** A `GlyphAmbiguous` answer should give `citation-symbol-ambiguous` naming the candidates' files and the "cite the declaring file" way forward. Card 4's `x.Foo` row reaches only the other ambiguity, a package base name shared by two directories. Card 3's `p#G` row stops at the index verdict. Evidence: `_lym/plan/04-citation-check.md` Tests rows; decision record D4 ("An ambiguous answer not split by build constraint is a `citation-symbol-ambiguous` finding naming the candidates' files").
- **D5, `envSeamStructs` is never checked.** The fixture rows pass `envStructs` as a literal `[]string{"p#Env","p#Inner"}`. No row asserts that the derived set holds a nested by-value struct, such as `internal/landingshed#Deps`, or drops a `nilLegal` path. If `envSeamStructs` returned only `internal/shedrecipe#Env`, every row and the repository scan would still pass. That leaves the criterion "markers missing a struct `Env` nests fails the scan" unguarded for the real set. Evidence: `_lym/plan/06-seam-marker-scan.md` fixture rows.
- **D5, two scan branches have no row.** No row covers a test calling `reflect.VisibleFields`. No row covers bare `NilSeams` inside package `envkit` (the repository scan meets it only on tests that are already marked, so a broken branch fails nothing). No row covers `NilSeams` reached through a same-package helper. Evidence: card 6 Intent bullets against its fixture rows; decision record D5.
- **D6, several covering and holding branches have no row.** Uncovered covering branches:
  - the test's file under `Uses`, `Delete`, `Rename` or `Move`;
  - the package self glyph of the test's unit or its `_test` counterpart;
  - the card's own `Edit` naming the test glyph;
  - a later card's `Extends` of the test.

  Uncovered holding branches: a card held through `Extends`, `Delete` or `Rename` targets. Evidence: `_lym/plan/08-seam-test-check.md` Tests rows.
- **D9, mode coverage exists for D7 only.** Card 7 drives `test-tier-unstated` through every mode: gate, lens gate, no-flag, `--lens-review`, `--overlay`, `--rework`, and `--require-approved` as a negative. Cards 8 and 9 only say "the gate and the verb both report". No row puts `seam-test-unlisted` or `doc-go-unlisted` through `--rework`, `--overlay` or `--lens-review`, or shows `--require-approved` not reporting them. No card has a row that webster's dispatch reports none of the three. Evidence: decision record D9 criterion; card 8 and card 9 `TestGateParity_PlanGate` rows.
- **D2/`PATTERN-refusal-way-forward`, way-forward text not asserted.** `TestCheckDecisionIDs` asserts only the Check and the id or heading in the Detail. The record's D2 criterion names "the way forward", and PATTERN requires a test that reaches each new refusal's text. The same applies to `citation-*`, `test-tier-unstated`, `seam-test-unlisted` and `doc-go-unlisted`: their rows count findings by check and never assert the way-forward clause. Evidence: card 1 Tests ("asserted by its exact Check and the id or heading"); card 7 ("counts findings by check").

**LOW**
- **D3, card reads HEAD's tree instead of what git tracks.** D3 says "an entry git tracks at the worktree root"; card 4 reads HEAD's tree. A root entry that is staged but not yet committed is tracked but goes untested. Card 4 also makes "a worktree with no commit" a returned error, and no row reaches that branch.
- **D8, branches without a row:**
  - the `TestDo` → `TestDoes` rename row satisfies both the "old side is test code" and the "name begins `Test`" conditions, so neither is reached alone;
  - a later card's `Extends` of the `doc.go`;
  - a `_test.go` file target or `_test` pseudo-unit counted as test code;
  - an ambiguous answer counted as non-test;
  - `Prosa` and `Custom` groups skipped;
  - a non-`Test` helper created by handle counting as non-test code.

  Evidence: card 9 Tests rows.
- **Card 4, darn form of the "discussion" gate:** no row shows it still resolves without `PlanIndex` or `WorktreeRoot`.
- **Card 1, untested details:** no row asserts the loom ordering (Validate, then decision ids, then citations, then relevant list), and no row covers an empty-title `### D1 —` heading.
- **D11, help text:** no row asserts that `validate-discussion --help` and `validate-plan --help` name the new checks, although the criterion says they do.
- **Card 3, `ResolveGlyphs` outage:** no row covers `ResolveGlyphs` returning an error wrapping `ErrQuarryUnavailable`. Card 4's fake covers only the loomshed side.
- **Outside my lens:** `envkit`'s `noFindingsIndex` embeds a nil `planindex.Index`. A test that runs the loom discussion gate through `FullEnv` over a record with a citation would panic on `ResolveGlyphs`. No card lists that fake.

No further tool calls were made.
