# Card 8 — shedrecipe-fixer-element

**Create:**
- `plan:internal/shedrecipe#GateFixerRequest` -> `type GateFixerRequest struct`
- `plan:internal/shedrecipe#splitGateFixer` -> `func splitGateFixer(entry string, cfg Config, env Env) (Config, *GateFixerRequest, error)`
- `plan:internal/shedrecipe#applyGateFixer` -> `func applyGateFixer(row string, spec shuttleengine.GateSpec, req *GateFixerRequest, env Env) (shuttleengine.GateSpec, error)`
- `plan:internal/shedrecipe#TestGateFixerElement` -> `func TestGateFixerElement(t *testing.T)`
- `plan:internal/loomshed#LensReviewOnly` -> `func LensReviewOnly(findings string) bool`
- `plan:internal/loomshed#TestLensReviewOnly` -> `func TestLensReviewOnly(t *testing.T)`

**Edit:**
- `internal/shedrecipe#Env`
- `internal/shedrecipe#darnWriteEntry`
- `internal/shedrecipe#planWriteEntry`
- `internal/shedrecipe#discussionWriteEntry`
- `internal/shedrecipe/doc.go`
- `contracts/specs/shed-recipe-spec.md`
- `internal/shedrecipe#TestResolveGateSpec_Rejections`
- `internal/shedrecipe#TestPlanWriteEntry_GateConfig`
- `internal/shedrecipe#TestDarnWriteEntry_HappyPath`
- `internal/shedrecipe#TestDiscussionWriteEntry_GateConfig`
- `contracts/recipes/darn-recipe.yaml`
- `contracts/recipes/tapestry-recipe.yaml`
- `internal/testkit/envkit#nilLegal`
- `internal/battencli#intentionallyNil`
- `internal/loomcli#intentionallyNil`

**Uses:**
- `internal/shedrecipe#resolveGateSpec`
- `internal/shedrecipe#resolveRowGateSpec`
- `internal/shedrecipe#planWriteGateSpec`
- `internal/shedrecipe#fakeSetting`
- `internal/shedrecipe#newTestEnv`
- `internal/shedrecipe#darnTestEnv`
- `internal/loomshed#ValidatePlanLensReview`
- `internal/loomshed/doc.go`
- `internal/testkit/envkit#TestFullEnv_FillsEverySeam`
- `internal/testkit/envkit#TestNilSeams_ReportsExactPaths`
- `internal/testkit/envkit#TestNilSeams_SkipsNilLegalPaths`
- `internal/testkit/envkit#TestFullEnv_PathFieldsAbsoluteUnderOneRoot`
- `internal/testkit/envkit#TestSeamMarkers_ReflectiveTestsAreMarked`
- `internal/battencli#TestWire_EverySeamFilled`
- `internal/loomcli#TestWire_EverySeamFilled`
- `internal/loomcli#TestWire_BeamRecipe`
- `internal/loomcli#TestWire_DarnRecipe`
- `internal/loomcli#TestWire_SamplerRecipe`

**Intent:**
Let a row gate element opt into a fixer (D3) and wrap the three fixer rows' gates once each row has finished its spec (D2).

- `GateFixerRequest` carries `Row`, `Entry` (the element's gate name), `Model` (the resolved model-spec), `Spec shuttleengine.GateSpec`, `Forked bool` and `SkipForked func(findings string) bool`.
- `Env` gains `GateFixer func(GateFixerRequest) (shuttleengine.GateSpec, error)`, the wrap seam card 9 wires; nil leaves every gate as today.
  It is nil-legal in `envkit`'s `nilLegal`, and `battencli`'s and `loomcli`'s `intentionallyNil` lists name it until card 9 removes it from `loomcli`'s.
- `splitGateFixer(entry, cfg, env)` reads an optional `fixer` key on each element of the row's `gates` list and returns `cfg` with that key removed from every element, so `resolveGateEntry`'s unknown-key check stays as it is and still refuses `fixer` on every row whose constructor does not call `splitGateFixer`.
  `fixer` names a `<module>.<key>` setting, looked up through `env.Setting` at build as the `parts` gate reads `tapestry.parts_max`.
  It refuses, naming the entry, the element index and the way forward: a `fixer` that is not a `<module>.<key>` string; a setting `env.Setting` cannot find (way forward: name a key of the module's config, or remove `fixer`); a setting holding a non-string; a non-empty value `modelspec.Parse` rejects (way forward: set the key to a model-spec with `lym config <module>`, or `""` to turn the fixer off); and `fixer` on two elements of one list.
  `env.Setting` is required only when an element carries `fixer`.
  An empty setting value returns a nil request: the fixer is off and the row behaves exactly as today.
- `applyGateFixer(row, spec, req, env)` returns `spec` unchanged for a nil request or a nil `env.GateFixer`, and otherwise calls `env.GateFixer` with the request's `Row` and `Spec` filled.
- `darnWriteEntry`, `planWriteEntry` and `discussionWriteEntry` call `splitGateFixer` on their row config before resolving `gates`, and `applyGateFixer` once their spec is complete: after Darn-Write sets every entry's `Reprompt`, and after `planWriteGateSpec` sets the `plan` entry's `Forked`, so the wrap sees the closures each row's constructor leaves.
  Plan-Write's request sets `Forked` when the `tapestry.plan_lenses` setting is non-empty, the same setting that gives its lead fork authorization, and `SkipForked` to `loomshed.LensReviewOnly` (D5).
- `LensReviewOnly(findings)` reports true when the forked plan gate's findings text, as `planGate` formats it, holds a blocking finding and every blocking finding is `lens-review-missing`; it lives beside `ValidatePlanLensReview`, which owns that check name and format.
- The recipes opt in: Darn-Write's `verify` element gains `fixer: darn.gate_fixer`, Plan-Write's `plan` element `fixer: tapestry.plan_gate_fixer`, and Discussion-Write's `discussion` element `fixer: loom.discussion_gate_fixer`; each row's recipe comment says what `fixer` does.
  Darn-Write's `clean` and `description` elements get no fixer of their own; the wrap makes them wait on a live fixer of the row.
- `doc.go` documents the `fixer` element key, the three rows that read it and the `GateFixer` seam.
- `contracts/specs/shed-recipe-spec.md`'s gate-capable row paragraph names the optional `fixer` element key: a `<module>.<key>` setting holding a model-spec, or `""` to turn the fixer off, read only by the Darn-Write, Plan-Write and Discussion-Write rows and refused as an unknown key on any other row.

Bound: only the three constructors read `fixer`; on any other row the key is the unknown-key refusal it is today.

Tests, tier0:
- `TestGateFixerElement` rows: no `fixer` key leaves the seam uncalled; an empty setting value leaves the seam uncalled; a nil seam leaves the spec unchanged; a nil `env.Setting` with an element carrying `fixer` is refused naming `Setting`; a missing setting, a non-string setting, an unparsable value, a malformed `fixer` value and `fixer` on two elements are each refused naming the entry, the element and the way forward.
  The seam call itself, with the row, the element's name, the model-spec and the complete spec, is pinned by the three row tests below, one per constructor.
- `TestResolveGateSpec_Rejections` gains a row: `fixer` on a gate element of a row that reads none is refused as an unknown key.
- `TestPlanWriteEntry_GateConfig` gains rows: with a recording seam, the request's spec carries the `plan` entry's `Forked` closure, `Forked` follows `tapestry.plan_lenses`, and `SkipForked` is set.
- `TestDarnWriteEntry_HappyPath` gains a row: the spec the seam receives carries the darn `Reprompt` on every entry and names `verify` as the fixer entry.
- `TestDiscussionWriteEntry_GateConfig` gains a row: the seam receives the `discussion` entry.
- `TestLensReviewOnly` rows: only `lens-review-missing` blocking is true; another blocking finding beside it is false; an informational finding beside it is still true; no blocking finding is false.

**ImpactSummary:** three row constructors and the recipes gain the opt-in, and `Env` gains one nil-legal seam whose seam tests are listed; every other gated row resolves exactly as today.
