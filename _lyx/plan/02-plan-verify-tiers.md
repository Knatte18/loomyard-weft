# Card 2 — plan-verify-tiers

**Edit:**
- `internal/loomengine#PlanTable` -> `func PlanTable(layout *lyxcwd.Location, dirs configengine.Dirs, stencilsDir, specsDir string, cfg Config, reg modelspec.Registry, verifyTiers []string) (seatengine.Table, error)`
- `internal/loomengine#TestPlanTable`
- `internal/loomengine#TestProducerSpecs_SkillsAndParentDirective`
- `internal/loomcli#loomCLI.wire`
- `internal/loomcli#TestWire_LanguageSkills`
- `contracts/stencils/loom/loom-template-plan.md`

**Uses:**
- `plan:internal/languageconfig#VerifyTiersValue`

**Intent:** The plan writer's prompt renders the hub's `verify_tiers` and stops naming this repository's Go tiers (D6), and its role line says it never asks the operator (D4).

`PlanTable` takes the told `verifyTiers` and fills a required `verify_tiers` marker in the table's `Values` with `languageconfig.VerifyTiersValue(verifyTiers)`; the marker is not added to `Optional`.
Its doc comment names the new value and that an empty list renders `none`.
`loomCLI.wire`'s `PlanTable` closure passes the `Language.VerifyTiers` Card 1 keeps.

`loom-template-plan.md`:

- The role line says the Plan producer never interviews and never asks the operator, that its only messages are the parent requests its Step 1 scope rules name, and that it has no review logic of its own.
- Card rule 1 drops `go build ./...`: the project builds with the repo's build immediately after the card's commit.
- The `## verify:` paragraph keeps its language-neutral rules: commands not prose, the plan-level section as the one integration check, the per-card `**Verify:**` package scope with the `verify-module-wide` and `lyx gate test` sentence (a Go-coded gate the Out list keeps), and the requirement that the section runs every targeted package's tests, its hermetic build-tagged tests included, with no `-tags integration` example.
  It then says the section runs the hub's verify tiers in the order and form this list gives, followed by `{{.verify_tiers}}` on its own line, and the fallback: with `none`, the section builds the code and then runs the tests of every package a card targets.
  The pointer to `verify-nested-module` in the spec stays, with no `go -C` form spelled out.
  The `go build`/`go vet`/`go test` order sentence, the `llm` sentence and the `tmux` sentence leave the stencil; Plan-Review still flags a targeted package the section leaves unrun, and the spec pointer for tier definitions stays.
- The header comment lists `verify_tiers` as a required marker filled by `PlanTable`, rendered `none` for an empty list.
- No `{{if}}`/`{{range}}` is added, and no range `webster-fork-stencil-fixes` cites (the `## Rename mechanic` block) is edited.

Tests:

- `TestPlanTable` gains a subtest over the rendered lead prompt (through `seatengine.Prompts`, as its rows do):
  with two tiers, the first naming a Go build step in backticks and the second holding `{{.x}}`, the prompt holds the two bullets consecutively and in order, verbatim, the `{{` entry included;
  with nil tiers the prompt holds `none` directly after the list sentence and holds no `- ` line with empty content.
  The ban on Go tier commands in the stencil text is Card 4's one scan over the hub-generic stencils, not an assertion here.
  Its existing rows pass nil, and their "no leftover `{{`" check is unaffected because nil renders `none`.
- `TestProducerSpecs_SkillsAndParentDirective` passes nil to `PlanTable`.
- `TestWire_LanguageSkills`'s listed row (Card 1 put tiers in its file) evaluates `c.env.PlanTable()` and asserts its `Values["verify_tiers"]` equals `languageconfig.VerifyTiersValue` of the file's tiers.

**ImpactSummary:** `PlanTable` re-signed with one production caller (the loom wire closure) and two tests; the plan stencil gains one required marker, filled at its only render site.
