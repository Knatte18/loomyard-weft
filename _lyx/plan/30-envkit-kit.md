# Card 30 — envkit-kit

**Create:**
- `testkit/envkit/envkit.go`
- `testkit/envkit/envkit_test.go`

**Uses:**
- `internal/shedrecipe#Env`
- `internal/landingshed#Deps`
- `internal/loomshed#PRReworkDeps`
- `testkit/shedfake/shedfake.go`

**Intent:**
Provide one builder each for a fully filled `shedrecipe.Env`, a `landingshed.Deps` and the loom recipe's rework deps (#324 §3), so a new required field touches the kit plus the two cycle-forced local builders instead of every copy.

`envkit.go` imports `shedrecipe`, `landingshed`, `loomshed`, `battenshed`, `websterengine`, `shuttleengine`, `shedengine`, `fabricengine` (for the typed-nil fabric opener's signature), `summaryparser` and `testkit/shedfake`; no `*cli` and no spawning package.

- `LandingDeps(dir string) landingshed.Deps`: every field the Publish/Finalize constructors require, filled with told values derived from dir, the typed-nil fabric opener for both openers and `shedfake.MergeShuttle` — the shape `shedbuild/fixture_test.go`'s and `loomrecipe/fixture_test.go`'s `testLandingDeps` share.
- `ReworkDeps(t testing.TB, dir string) loomshed.PRReworkDeps`: the rework deps with its directories created under dir and no-op closures, the shape `shedbuild`'s inline literal and `loomrecipe`'s `testReworkDeps` share.
- `FullEnv(t testing.TB) shedrecipe.Env`: every path field an absolute path under one `t.TempDir()`, every seam filled — `shedfake.Shuttle`, `shedfake.BurlerRunner`, `shedfake.WebsterSeams()`, a zero-result `WebsterRun`, the spec closures, `LandingDeps`, `ReworkDeps`, the rework spec and the batten fields — modelled on `shedbuild/fixture_test.go`'s `newTestEnv`, the fullest copy.
  A seam any registered engine requires non-nil must be filled here.

- `NilSeams(env shedrecipe.Env) []string`: walks `env` with `reflect`, recursing into nested structs (`WebsterDeps`, `Landing`, `Rework`, `SeedChild`, `InnerRun`, `Teardown`, `PrimeLock` and any added later), and returns the dotted path of every func, interface, pointer, map or slice-of-func field that is nil.
  It reports and never asserts, so the D3 no-assertion rule holds; the CLI wiring tests (cards 34, 35) and this kit's own test assert on its answer.

Callers mutate the returned value; there are no option structs.
The builders `t.Fatalf` only on their own setup failures (D3).
The package doc names the cycle-forced exceptions: `shedrecipe` and `landingshed` keep one local builder each.

`envkit_test.go` asserts that `NilSeams(FullEnv(t))` is empty, so a newly added seam fails here until the kit fills it; that `NilSeams` reports the exact path of a seam set to nil at the top level and inside a nested struct; and that every path field of `FullEnv` is absolute and under one temp root.
