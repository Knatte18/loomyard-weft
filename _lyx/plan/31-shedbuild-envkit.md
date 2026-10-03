# Card 31 — shedbuild-envkit

**Edit:**
- `shedbuild/fixture_test.go`

**Uses:**
- `testkit/envkit/envkit.go`
- `testkit/shedfake/shedfake.go`

**Intent:**
Replace shedbuild's private copy of the full `Env` builder and its fakes with the kits (#324 §2, §3).

Delete `fakeShuttle`, `evalGateList`, `fakeBurlerRunner`, `fakeWebsterRun`, the four placeholder seams, `fakeMergeShuttle`, `nilFabricOpener`, `testLandingDeps` and `newTestEnv` from `fixture_test.go`; callers use `envkit.FullEnv(t)` (and `envkit.LandingDeps` where they built only landing deps).
`TestBuild_EveryRegisteredEngineBuilds` keeps its guarantee that every registered engine builds from the fixture, now against `FullEnv`.
`writeStencilFile` is package-specific and stays.
The file's header comment, which justifies the copy by package reachability, is rewritten to describe what remains.

The told-geometry property (no path outside the test's own temp dir) holds because `FullEnv` derives every path from one `t.TempDir()`.
Apply D10 to every touched test.

**ImpactSummary:** One untagged shedbuild fixture file loses its copied builders and fakes; every shedbuild test keeps building from an equivalent full Env.
