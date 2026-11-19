# Card 15 — finalize-reap

**Edit:**
- `internal/landingshed#Deps`
- `internal/landingshed#Finalize.Call`
- `internal/landingshed#TestFinalize_StuckReasons`
- `internal/landingshed#TestFinalize_MergeMarkPushOrderAndVerdict`
- `internal/loomcli#landingDeps`
- `internal/loomcli#TestLandingDeps_EveryFieldPopulated`
- `plan:internal/loomcli#TestWire_HardenerRecipe`
- `internal/testkit/envkit#nilLegal`
- `internal/testkit/envkit#FullEnv`

**Extends:**
- `internal/landingshed/doc.go`

**Uses:**
- `internal/loomcli/doc.go`
- `plan:internal/loomcli#HardenReapDeps`
- `internal/battencli#TestWire_EverySeamFilled`
- `internal/loomcli#TestWire_BeamRecipe`
- `internal/loomcli#TestWire_DarnRecipe`
- `internal/loomcli#TestWire_EverySeamFilled`
- `internal/loomcli#TestWire_SamplerRecipe`
- `internal/testkit/envkit#TestFullEnv_FillsEverySeam`
- `internal/testkit/envkit#TestNilSeams_ReportsExactPaths`
- `internal/testkit/envkit#TestNilSeams_SkipsNilLegalPaths`

**Intent:**
Finalize reaps the sandbox root before it lands (D5), so the worktree's teardown meets no sandbox hub.

`landingshed.Deps` gains `Reap func() error`, nil for every recipe but hardener.
`Finalize.Call` calls it once, after the change description parses and before it takes the parent lock and merges.
A failing reap halts Finalize Stuck with the reap's error and "way forward: run `lym harden sandbox clear` in the task worktree, then `lym loom resume`".
`landingDeps` fills it, for a run whose recipe holds a `HardenFix` row, with `hardenshed.Reap` at `ReapClear` over `loomcli.HardenReapDeps`.
`envkit`'s `nilLegal` gains `Landing.Reap` and `FullEnv` fills it, so the seam tests over every other recipe's wiring keep passing.

`TestFinalize_MergeMarkPushOrderAndVerdict` gains a row: a set `Reap` runs before the merge, and a nil one is skipped.
`TestFinalize_StuckReasons` gains a row: a failing `Reap` halts Stuck naming the way forward and merges nothing.
`TestLandingDeps_EveryFieldPopulated` lists `Reap` among the fields a non-hardener recipe leaves nil, and `TestWire_HardenerRecipe` asserts the hardener wiring's landing deps carry a `Reap`.

**ImpactSummary:** One optional Finalize seam, nil for every recipe but hardener, so every other run lands exactly as before.
