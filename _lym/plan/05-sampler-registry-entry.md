# Card 5 — sampler-registry-entry

**Create:**
- `plan:internal/shedrecipe#samplerSessionEntry` -> `func samplerSessionEntry(row RowIdentity, cfg Config, env Env) (shedengine.ShedProducer, error)`
- `plan:internal/shedrecipe#TestSamplerSessionEntry` -> `func TestSamplerSessionEntry(t *testing.T)`

**Edit:**
- `internal/shedrecipe#registry`
- `internal/shedrecipe#LoomBuildEngines`
- `internal/shedrecipe#Env`
- `internal/shedrecipe#newTestEnv`
- `internal/shedrecipe#minimalEntries`
- `internal/shedbuild#checkLoomContract`
- `internal/shedbuild#TestBuild_LoomContract`
- `internal/testkit/envkit#FullEnv`
- `internal/testkit/envkit#nilLegal`
- `internal/testkit/envkit#TestFullEnv_FillsEverySeam`
- `internal/testkit/envkit#TestNilSeams_ReportsExactPaths`
- `internal/testkit/envkit#TestNilSeams_SkipsNilLegalPaths`
- `internal/testkit/envkit#TestFullEnv_PathFieldsAbsoluteUnderOneRoot`
- `internal/loomcli#intentionallyNil`
- `internal/loomcli#TestWire_EverySeamFilled`
- `internal/loomcli#TestWire_BeamRecipe`
- `internal/loomcli#TestWire_DarnRecipe`
- `internal/battencli#intentionallyNil`
- `internal/battencli#TestWire_EverySeamFilled`

**Extends:**
- `internal/shedrecipe/doc.go`

**Uses:**
- `internal/shedrecipe#prReviewEntry`
- `internal/testkit/envkit#TestSeamMarkers_ReflectiveTestsAreMarked`
- `internal/shedrecipe#darnWriteEntry`

**Intent:**
D1 and `PATTERN-shed-recipe-registry`: `SamplerSession` is one new entry of the one `registry` map, built by `samplerSessionEntry`, and joins `LoomBuildEngines`, so a recipe whose build row runs it passes the loom contract's build-row clause.
`Env` gains `Sampler loomshed.SamplerDeps`, a whole-struct passthrough read by the SamplerSession entry only, documented as `Darn` is.
`samplerSessionEntry` refuses every config key with a way forward naming the key's removal, as `prReviewEntry` does, refuses each nil seam of `Env.Sampler` but `Watch` by field name through `requireSeam` and each empty record path through `requireAbsRoot`, and returns `loomshed.NewSamplerSession(row.Name, env.Sampler)`.
`checkLoomContract`'s engine-choice text becomes `Implementer, DarnWrite, BeamAwait or SamplerSession`, and `TestBuild_LoomContract`'s two expected way-forward strings follow it; `contracts/specs/shed-recipe-spec.md` names the set as `shedrecipe.LoomBuildEngines` and needs no change.
`TestSamplerSessionEntry` holds only what `TestEveryEntry_RejectsUnknownConfigKey` does not already reach through the new `minimalEntries` row (a built producer and the refused `bogus_key`): a nil `Sampler.Stamp` and a nil `Sampler.Sessions` are each refused naming the field, an empty `Sampler.RecordPath` is refused naming it, and a nil `Sampler.Watch` builds.
`minimalEntries` gains a `SamplerSession` row with empty config, so `TestEveryEntry_RejectsUnknownConfigKey` covers the entry, and `newTestEnv` and `envkit.FullEnv` fill `Sampler` with absolute paths under their one root and no-op seams; `nilLegal` gains `Sampler.Watch`.
Every test marked as reflecting over `shedrecipe#Env` gains the marker line `//lym:seam internal/loomshed#SamplerDeps`: the four envkit tests, the three loomcli wire tests and battencli's `TestWire_EverySeamFilled`.
`wire` fills no sampler seam for a tapestry seed and batten runs no sampler row, so loomcli's `intentionallyNil` lists each func seam of `Sampler` but `Watch` as "sampler-only seam, filled by wire for a sampler seed", and battencli's as "sampler-only seam, batten runs no sampler row".
`shedrecipe/doc.go` gains a `# The SamplerSession row` section saying it takes no config key, reads `Env.Sampler` and is a loom build engine.

**ImpactSummary:** registers one engine and one Env field; every Env-reflecting seam test and fixture builder in shedrecipe, envkit, loomcli and battencli is updated, and the contract refusal text changes in one test.

**Commit:** 5: shedrecipe: register the SamplerSession build engine
