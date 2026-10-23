# Card 19 — the arbiter routes a plan amendment

**Edit:**
- `internal/loomcli#arbiterPromptInput`
- `internal/loomcli#arbiterPrompt`
- `internal/loomcli#loomCLI.arbiterDepsFor`
- `internal/loomcli#TestArbiterPrompt`
- `internal/loomcli#arbiterDeps`
- `internal/loomcli#arbiterCall.settleFresh`
- `internal/loomcli#TestArbiterVerb_SpawnPath`
- `internal/loomcli#arbiterFixture`
- `internal/loomcli#arbiterFixture.deps`
- `internal/loomcli/doc.go`
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
`arbiterDeps` gains `AmendmentTarget func() string`, read at each spawn and never at wiring time: the Go driver builds its `arbiterDeps` once and holds them for the whole drive, and the plan is approved mid-drive, so a value read at wiring time would stay `none` for every later stop of that drive.
`arbiterCall.settleFresh` sets `promptInput.AmendmentTarget` from `c.deps.AmendmentTarget()` beside the other per-attempt inputs it fills after copying `Prompt`.
`arbiterDepsFor` reads `recipeTarget, err := loomrecipe.AmendmentTarget(c.recipe)` and returns `err` like its other build errors, a fail-loud wiring guard that takes no reaching row, since only a broken embedded recipe or a recipe name outside the three can raise it;
it then sets `AmendmentTarget` to a closure over `arbiterAmendmentTarget(recipeTarget, planApprovedReader(location.AnchorPath()))`, the same approval `--amends-plan`'s refusal reads.
The generic step envelope the driver saves is unchanged.

`loom-template-arbiter.md`: its header comment lists `{{.amendment_target}}` (a row name or the literal `none`), the inputs section names it, "What you may change" adds `--amends-plan` to the `decision add` line, and a new paragraph says: when `amendment_target` names a row, a stop whose cause is a plan defect that the board entry and the decision record already settle is your call: record the amendment with `lyx loom decision add {{.slug}} --by arbiter … --amends-plan`, wording it as a concrete change to named cards, each named by slug as well as number, and write a `resume` verdict whose `goto` is that target.
The hand-on rule is unchanged: an amendment that needs an operator decision or widens scope is handed on.

`internal/loomcli/doc.go`'s Arbiter section gains a paragraph: the prompt carries `amendment_target`, which is the run's recipe's `amendment_target` key (loom's `Plan-Burler-Review`; darn and batten set none) once the current generation's plan is approved and `none` otherwise, read at each spawn so a drive that began before approval still routes; the arbiter may then record a plan amendment and name that target as its goto.

What it lets through, and the bound: each `--amends-plan` call re-admits one goto onto the target; the arbiter's repeats are capped by `arbiter_max_spawns`, counted from its attempt records, and scope widening is bounded by the hand-on rule, since the arbiter's entry joins the answer key Plan-Review and PR-Gate's alignment agent judge against.

Tests:
`TestArbiterPrompt` gains rows: a set `AmendmentTarget` renders into the prompt, and an empty one renders `none`.
`TestArbiterAmendmentTarget` rows: loom's target with an approved plan answers `Plan-Burler-Review`; an unapproved plan, an approval read error and an empty recipe target (darn) answer `none`.
`arbiterFixture` gains `amendmentTarget func() string`, and `arbiterFixture.deps` sets `AmendmentTarget` from it, answering `none` when a row sets none.
`TestArbiterVerb_SpawnPath` gains a row: a fixture whose `amendmentTarget` answers `Plan-Burler-Review` only once the attempt record exists under `f.attemptDir(1)`, and `none` before, writes a prompt naming `Plan-Burler-Review`, which pins the read after `settleFresh` writes the attempt record, at the spawn.
The row builds its own deps and so never reaches `arbiterDepsFor`; that `arbiterDepsFor` sets a closure and never calls it is held by its code alone.

**ImpactSummary:** `arbiterPrompt` has one caller, the arbiter call's spawn, whose input gains one field; `arbiterDeps` is built by `arbiterDepsFor` for both the verb and the driver, and each spawn now reads the target afresh; the stencil gains one required marker that `arbiterPrompt` always fills.
