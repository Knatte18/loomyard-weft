# Card 24 — arbiter-names-run-dir

**Edit:**
- `internal/loomcli#arbiterRecipeDescription` -> `func arbiterRecipeDescription(recipe, implementerNote string) string`
- `internal/loomcli#loomCLI.arbiterDepsFor`
- `internal/loomcli#TestRecipeKeyedWiring`

**Create:**
- `plan:internal/weaverengine#ArbiterNote` -> `func ArbiterNote(impl Implementer, anchorRoot string) string`
- `plan:internal/weaverengine#TestArbiterNote` -> `func TestArbiterNote(t *testing.T)`

**Extends:**
- `internal/weaverengine/doc.go`

**Uses:**
- `internal/weaverengine#ActiveOrSetting`
- `internal/loomcli/doc.go`

**Intent:** The arbiter's description of a tapestry run names the active implementer's run directory, so a stop on any row reaches the plan, state, reports and journal (D8).

- `ArbiterNote` renders, for Weaver, the plan directory, the state file, the reports directory and `journal.md`; for Webster, its run directory and `state.json`.
- `arbiterRecipeDescription` takes the note and puts it after the run description of a recipe with a `PlanWrite` row, before the repair verbs; other recipes ignore it.
- `arbiterDepsFor` computes the note from the seam's advisory read (`ActiveOrSetting`) for the run's anchor, logging a fallback's reason.

`TestArbiterNote` is tier0: Weaver's note names the plan directory, `state.json`, the reports directory and `journal.md` under its anchor; Webster's names `_lym/webster` and `state.json`.
`TestRecipeKeyedWiring`'s Tapestry row passes a note and checks the description carries it before the repair verbs; the other recipes' rows pass a note and check it is absent.

**ImpactSummary:** Re-signs one unexported loomcli helper with its one caller; descriptions of non-tapestry recipes are unchanged.
