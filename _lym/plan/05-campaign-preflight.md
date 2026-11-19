# Card 5 — campaign-preflight

**Create:**
- `plan:internal/hardenshed#CampaignCheck` -> `type CampaignCheck struct`
- `plan:internal/hardenshed#CheckCampaign` -> `func CheckCampaign(in CampaignCheck) error`
- `plan:internal/hardenshed#Rotate` -> `func Rotate[T any](list []T, rounds int) []T`
- `plan:internal/hardenshed#TestCheckCampaign` -> `func TestCheckCampaign(t *testing.T)`
- `plan:internal/hardenshed#TestRotate` -> `func TestRotate(t *testing.T)`
- `plan:internal/preflightshed#TestPreflight_HardenCheck` -> `func TestPreflight_HardenCheck(t *testing.T)`

**Edit:**
- `internal/preflightshed#Checks`
- `internal/preflightshed#preflightProducer.Call`
- `internal/shedrecipe#Env`
- `internal/shedrecipe#preflightEntry`
- `internal/testkit/envkit#nilLegal`
- `internal/testkit/envkit#FullEnv`

**Extends:**
- `internal/hardenshed/doc.go`
- `internal/preflightshed/doc.go`

**Uses:**
- `internal/impactset#HoldsTierTest`
- `plan:internal/gateslot#HubSetsGate`
- `plan:internal/gateslot#BilledTier`
- `internal/battencli#TestWire_EverySeamFilled`
- `internal/loomcli#TestWire_BeamRecipe`
- `internal/loomcli#TestWire_DarnRecipe`
- `internal/loomcli#TestWire_EverySeamFilled`
- `internal/loomcli#TestWire_SamplerRecipe`
- `internal/testkit/envkit#TestFullEnv_FillsEverySeam`
- `internal/testkit/envkit#TestFullEnv_PathFieldsAbsoluteUnderOneRoot`
- `internal/testkit/envkit#TestNilSeams_ReportsExactPaths`
- `internal/testkit/envkit#TestNilSeams_SkipsNilLegalPaths`
- `internal/testkit/envkit#TestSeamMarkers_ReflectiveTestsAreMarked`

**Intent:**
Preflight's refusals of a `hardener` run (D2) and the cyclic model rotation (D3).

`Rotate` expands a list cyclically to `rounds` entries, entry i being `list[i % len(list)]`; an empty list stays empty.
With a three-entry list and 10 rounds, rounds 1 to 9 take entries 1, 2, 3, 1, 2, 3, 1, 2, 3.
Card 14 hands the expanded reviewer and fixer lists to burler's per-round model lists, so a round past the list's length never clamps to its last entry.

`CampaignCheck` carries the loaded `Config`, the worktree root, the tiers each live key's run tests (the key's tiers with each tier's `covers`, as `lym gate test` narrows `{test_targets}`), whether the hub sets each key (`gateslot.HubSetsGate`), a told `HoldsTierTest(dir string, tiers []int) bool`, wired to `impactset.HoldsTierTest`, and a told `Route func(key string) (string, error)`, which card 14 wires to `reviewbudget.Route` over `hardenshed.ConfigSpec` so a refusal names the write for the layer the key's value comes from.
Copies run when `llm_driving` is false and `concurrency` is above 1.
`CheckCampaign` returns the first refusal in this order, each naming its way forward and ending "then run `lym loom resume`":
1. `packages` empty, or naming a directory the worktree lacks: names `Route("packages")`, which for a value from the pair's local layer is `lym config hardener --layer local --set packages=…` in the task worktree with the matching `run_settings` correction through `lym board upsert`.
2. The hub sets no `harden-live` key, or no `harden-live-copies` key when copies run: names `lym config gate --set 'gates.<key>={tiers: [<live tier>], scope: diff}'` from the prime.
3. The live key, or the copies key when copies run, names `gateslot.BilledTier`: names the same `lym config gate --set` write.
4. A package holds no test of a tier the `harden-live` key's run tests, or, when copies run, of a tier the copies key's run tests: names the package, the key, `Route("packages")` and the `lym config gate --set` write of the key.
5. A `reviewer` list of one entry, or one where two neighbours of its cyclic expansion are one model, its last and first entries included: names `Route("reviewer")` with a list without such a pair.
6. `max_bounces` above `hard_max_bounces`: names `Route("max_bounces")`.

`preflightshed.Checks` gains `Harden func() error`, run after the existing checks with their failure prefix; `shedrecipe.Env` gains `PreflightHarden func() error`, which `preflightEntry` passes as `Checks.Harden`; nil skips it, as for every other recipe.
Card 14 fills `PreflightHarden` for a recipe holding a `HardenFix` row.
`envkit`'s `nilLegal` gains `PreflightHarden`, as it holds `PreflightDrift`, and `FullEnv` fills it, so the seam tests over every other recipe's wiring and `NilSeams(FullEnv(t))` keep passing.

`TestCheckCampaign` is tier0, one row per refusal above, its input the only defect, each asserting the way forward and the resume, plus a passing campaign; the tier rows run a fake `HoldsTierTest`, and one row passes a package whose only test tier the key reaches through `covers`; the route rows run a fake `Route`.
`TestRotate` is tier0: the nine-round row above, a one-entry list and an empty list.
`TestPreflight_HardenCheck` is tier0: a failing `Harden` check halts Preflight Stuck with the check's text, a nil one is skipped, a passing one passes.

**ImpactSummary:** One optional check field on Preflight and one Env field, nil for every recipe but hardener and listed nil-legal in envkit; the check itself is new code in hardenshed.
