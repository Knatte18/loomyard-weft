# Card 18 — validate-parts-verbs

**Create:**
- `plan:internal/loomcli#loomCLI.validatePartsCmd` -> `func (c *loomCLI) validatePartsCmd() *cobra.Command`
- `plan:internal/loomcli#verbRowEngines` -> `var verbRowEngines map[string]string`
- `plan:internal/loomcli#TestValidatePartsCmd` -> `func TestValidatePartsCmd(t *testing.T)`
- `plan:internal/loomcli#TestValidatePlanCmd_Part` -> `func TestValidatePlanCmd_Part(t *testing.T)`

**Edit:**
- `internal/loomcli#refuseVerbOffRecipe`
- `internal/loomcli#verbUsesLightweightWiring`
- `internal/loomcli#loomCLI.validatePlanCmd`
- `internal/loomcli#Command`
- `internal/loomcli#TestArmAt_RefusesOffLoomRecipes`
- `internal/loomcli#TestValidatePlanCmd_ModeFlagsAreExclusive`
- `internal/loomcli/doc.go`
- `docs/overview.md`

**Uses:**
- `plan:internal/partsparser#Validate`
- `plan:internal/loomshed#ValidatePart`
- `plan:internal/loomshed#ErrPartUnknown`
- `internal/loomcli#planVerbs`
- `internal/loomcli#TestValidatePlanCmd`

**Intent:** Give the Parts-Write agent and the part writers their self-checks (D3, D7): `lym loom validate-parts` and `lym loom validate-plan --part <name>`.

**Recipe refusal by rows.** `verbRowEngines` maps each verb that needs a row to the engine of that row: every verb of `planVerbs` to `PlanWrite`, and `validate-parts` to `PartsWrite`.
`refuseVerbOffRecipe` refuses a verb whose engine `loomrecipe.HasEngine` does not find in the run's recipe, read from the recipe's rows, never its name; the plan verbs keep their refusal text unchanged, and `validate-parts` is refused with "loom: validate-parts does not apply to this run, whose %q recipe has no Parts-Write row and so writes no parts file; way forward: none is needed, since there is nothing to check, and a run with a plan checks it with \"lym loom validate-plan\"".
`TestArmAt_RefusesOffLoomRecipes` gains the `validate-parts` row on the darn recipe, which has no `Parts-Write` row, and its admission on tapestry.

**validate-parts.** `validatePartsCmd` takes no arguments and runs on the lightweight wiring, which `verbUsesLightweightWiring` adds it to: it loads tapestry.yaml through `loomengine.LoadTapestryConfig` for `parts_max`, then calls `partsparser.Validate(c.env.AnchorPath, c.env.DecisionRecordPath, max)`, the same function the `parts` gate calls, and prints one envelope: `ok` with `parts_path` when there is no finding, else an error envelope `loom: parts file is not yet valid` with the findings under `findings`, one `check[/part]: detail` each; a config or read error is an error envelope carrying it.
It is read-only, and `Command` registers it beside `validate-plan`.
`TestValidatePartsCmd` (tier0) covers a valid parts file passing, a file with a check finding failing with that finding, an absent parts file failing with `parts-missing`, a hub whose tapestry.yaml holds an invalid `parts_max` answering an error envelope carrying the config error, and the same findings as `loomshed.NewPartsGate` reports for the same file.

**validate-plan --part.** `validatePlanCmd` gains `--part <name>`, a sixth mode exclusive with the four mode flags: combined with any of them it is refused before the plan is read, naming the other flag and the way forward to run `--part` alone.
It adds no recipe check of its own: `refuseVerbOffRecipe` already refuses `validate-plan` on every recipe without a `PlanWrite` row, and the only recipe holding one, tapestry, also holds the `PartsPlanWrite` row.
On a tapestry run that did not split, the parts file is absent, and the absent-parts-file refusal of `ValidatePart` answers.
It runs `loomshed.ValidatePart(c.env.AnchorPath, c.env.WorktreeRoot, name, c.env.PlanIndex, pc)` with the plan context the default mode builds; an `ErrPartUnknown` error is refused with its text, which names the parts the file holds, and the findings envelope and exit code follow the default mode's, `plan_dir` naming the part directory.
The help text's mode list gains `--part`, and `TestValidatePlanCmd_ModeFlagsAreExclusive` gains its four combinations.
`TestValidatePlanCmd_Part` (tier0, over a parts file and part directories in a hub fixture) covers a part using a dependency's export passing, a part with a dangling handle failing with `handle-dangling`, a card targeting a spot failing with `part-spot-target`, an unknown part name refused naming the held parts, and an absent parts file refused with its way forward.

The loomcli doc and `docs/overview.md`'s loom verb list and validate-plan sentences gain both.

**ImpactSummary:** One new read-only verb and one validate-plan mode; the plan verbs' refusal and the other five modes are unchanged.
