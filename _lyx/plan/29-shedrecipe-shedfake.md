# Card 29 — shedrecipe-shedfake

**Edit:**
- `shedrecipe/fixture_test.go`
- `shedrecipe/entries_batten_test.go`
- `shedrecipe/entries_bouncer_test.go`
- `shedrecipe/entries_burler_test.go`
- `shedrecipe/entries_describe_test.go`
- `shedrecipe/entries_discussionwrite_test.go`
- `shedrecipe/entries_gate_test.go`
- `shedrecipe/entries_planwrite_test.go`
- `shedrecipe/entries_prrework_test.go`
- `shedrecipe/entries_simple_test.go`
- `shedrecipe/entries_singlellm_test.go`
- `shedrecipe/env_test.go`
- `shedrecipe/registry_test.go`

**Uses:**
- `testkit/shedfake/shedfake.go`

**Intent:**
Move shedrecipe's in-package tests onto `shedfake` and leave the package exactly one local `Env` builder and one local `landingshed.Deps` builder (#324 §2, §3).

`envkit` will import `shedrecipe`, so shedrecipe's own tests cannot use it (D1); `shedfake` imports nothing that imports shedrecipe, so its fakes are usable here.

- `fixture_test.go`: `fakeShuttle`, `fakeBurlerRunner` and the four embed-the-interface placeholders go; `newTestEnv` fills `Shuttle`, `Burler` and `WebsterDeps` from `shedfake.Shuttle`, `shedfake.BurlerRunner` and `shedfake.WebsterSeams()`.
  `newTestEnv` stays the package's one `Env` builder.
- `entries_batten_test.go`: `newTestEnvWithSeedChild` becomes `newTestEnv` plus the seed-child mutation at its callers, or a thin wrapper that mutates `newTestEnv`'s result.
- `entries_simple_test.go`: `fakeLandingShuttle` becomes `shedfake.MergeShuttle`; `validLandingDeps` and `validGateDeps` merge into one local landing builder whose callers mutate the gate variant; `nilFabricOpener` stays as the one local copy.
- `entries_discussionwrite_test.go`'s `attachingShuttle` and `entries_bouncer_test.go`'s `judgeSeamFakeShuttle` become `shedfake.Shuttle` settings (`AttachFound`/`AttachResult`, `RunFn`) rather than wrapper types.
- Every `Call` site followed by the standard fatal-on-error and outcome-compare block uses `shedfake.RequireOutcome`/`shedfake.CallOK`.

Every scripted value and assertion stays.
Apply D10 to every touched test.

**ImpactSummary:** Untagged shedrecipe tests only; local seam fakes move to shedfake and duplicate Env/Deps builders collapse to one each, assertions unchanged.
