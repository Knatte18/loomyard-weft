# Card 21 — Pin the edit directive on every opening stencil

**Create:**
- `plan:contracts/stencils#editRulePhrases` -> `var editRulePhrases []string`
- `plan:contracts/stencils#editRuleSentences` -> `func editRuleSentences(body string) []string`
- `plan:contracts/stencils#TestStencils_OpeningStencilsRenderEditDirective` -> `func TestStencils_OpeningStencilsRenderEditDirective(t *testing.T)`
- `plan:contracts/stencils#TestStencils_NoEditRuleRestatement` -> `func TestStencils_NoEditRuleRestatement(t *testing.T)`

**Edit:**
- `contracts/stencils#openingStencilViolations` -> `func openingStencilViolations(opening map[string][]string, bodies map[string]string, marker string) []string`
- `contracts/stencils#TestStencils_OpeningStencilsRenderParentDirective`
- `contracts/stencils#TestParentDirectiveScans_SyntheticRows`
- `contracts/stencils#roleOpeningStencils`
- `PATTERN.md`

**Uses:**
- `plan:internal/editdirective#MarkerName`
- `contracts/stencils#registeredBodies`
- `contracts/stencils#askOperatorPhrases`

**Intent:**
Decision 14's enforcement: a stencil scan like the parent-directive one, extending its scans rather than adding a parallel suite (`PATTERN-test-economy`).
It fails when a spawned role's opening stencil lacks the edit-directive marker, or another stencil restates the rule (`PATTERN-producer-pointer-rule`).
It lands after cards 16 to 20, which put the marker on every opening stencil.

`openingStencilViolations` takes the marker it checks for as a parameter instead of building the parent-directive marker itself, and names that marker in each violation.
`TestStencils_OpeningStencilsRenderParentDirective` passes `{{.parent_directive}}`.
The new `TestStencils_OpeningStencilsRenderEditDirective` runs the same scan over the same `roleOpeningStencils` map with `{{.` + `editdirective.MarkerName` + `}}`.
`roleOpeningStencils`' comment says each opening stencil carries both markers, enforced in `parentdirective_test.go`.

`editRulePhrases`, declared in `stencils.go` beside `askOperatorPhrases`, is the closed list of phrases distinctive to the rule's statement: `` `sed` ``, `heredoc`, `replace_all` and `edit or write`, matched case-insensitively.
`editRuleSentences` returns each sentence of a body that contains one of them.
`TestStencils_NoEditRuleRestatement` fails for any registered stencil other than `edit-directive` with such a sentence, naming the stencil and the sentence; after card 17's Master edit no stencil has one.

`TestParentDirectiveScans_SyntheticRows` gains rows:
- An opening stencil lacking `{{.edit_directive}}` is one violation naming it.
- A body that says "rewrite it with a heredoc" yields one restating sentence.
- A body with none of the phrases yields none.
Each new guard test carries the `//testtiming:keep` and `//lyx:guard` lines its parent-directive siblings carry.

`PATTERN.md` gains, under "Agents and prompts" after `PATTERN-parent-directive`, a `PATTERN-edit-directive` entry marked (test): writing a spawned role's opening stencil means it renders the edit directive through `internal/editdirective`, and the `edit-directive` stencil is the no-script edit rule's only statement; the orch stencils are outside the rule.

**ImpactSummary:** Test-only re-sign of one scan helper with its two callers in the same file, plus one new PATTERN entry; no production code changes.

**Commit:** 21: stencils: pin the edit directive on every spawned role's opening stencil
