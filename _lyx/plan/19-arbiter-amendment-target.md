# Card 19 — the arbiter routes a plan amendment

**Edit:**
- `internal/loomcli#arbiterPromptInput`
- `internal/loomcli#arbiterPrompt`
- `internal/loomcli#loomCLI.arbiterDepsFor`
- `internal/loomcli#TestArbiterPrompt`
- `contracts/stencils/loom/loom-template-arbiter.md`

**Create:**
- `plan:internal/loomcli#arbiterAmendmentTarget` -> `func arbiterAmendmentTarget(recipeTarget string, approved func() (bool, error)) string`
- `plan:internal/loomcli#TestArbiterAmendmentTarget` -> `func TestArbiterAmendmentTarget(t *testing.T)`

**Uses:**
- `plan:internal/loomrecipe#AmendmentTarget`
- `plan:internal/loomcli#planApprovedReader`

**Intent:**
D8: the arbiter prompt gains an input, `amendment_target`, rendered beside the envelope path; the arbiter stencil stays recipe-blind.

`arbiterPromptInput` gains `AmendmentTarget`; `arbiterPrompt` fills the stencil's new required marker `amendment_target` with it, or the literal `none` when it is empty, as the stencil's other absent inputs are.
`arbiterAmendmentTarget(recipeTarget, approved)` returns `recipeTarget` when it is non-empty and `approved` answers true, and `none` otherwise; an `approved` error answers `none`, since a target the arbiter cannot rely on is no target.
`arbiterDepsFor` fills `Prompt.AmendmentTarget` from `arbiterAmendmentTarget(loomrecipe.AmendmentTarget(c.recipe == shedrun.RecipeDarn), planApprovedReader(location.AnchorPath()))`, the same approval `--amends-plan`'s refusal reads; a recipe-read error, which only a broken embedded recipe can raise, is returned from `arbiterDepsFor` like its other build errors, a fail-loud wiring guard that takes no reaching row.
The generic step envelope the driver saves is unchanged.

`loom-template-arbiter.md`: its header comment lists `{{.amendment_target}}` (a row name or the literal `none`), the inputs section names it, "What you may change" adds `--amends-plan` to the `decision add` line, and a new paragraph says: when `amendment_target` names a row, a stop whose cause is a plan defect that the board entry and the decision record already settle is your call: record the amendment with `lyx loom decision add {{.slug}} --by arbiter … --amends-plan`, wording it as a concrete change to named cards, each named by slug as well as number, and write a `resume` verdict whose `goto` is that target.
The hand-on rule is unchanged: an amendment that needs an operator decision or widens scope is handed on.

What it lets through, and the bound: each `--amends-plan` call re-admits one goto onto the target; the arbiter's repeats are capped by `arbiter_max_spawns`, counted from its attempt records, and scope widening is bounded by the hand-on rule, since the arbiter's entry joins the answer key Plan-Review and PR-Gate's alignment agent judge against.

Tests:
`TestArbiterPrompt` gains rows: a set `AmendmentTarget` renders into the prompt, and an empty one renders `none`.
`TestArbiterAmendmentTarget` rows: loom's target with an approved plan answers `Plan-Burler-Review`; an unapproved plan, an approval read error and an empty recipe target (darn) answer `none`.

**ImpactSummary:** `arbiterPrompt` has one caller, the arbiter call, whose input gains one field; the stencil gains one required marker that `arbiterPrompt` always fills.
