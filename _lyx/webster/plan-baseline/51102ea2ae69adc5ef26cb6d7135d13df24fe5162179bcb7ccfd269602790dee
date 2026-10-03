# Card 35 — loomcli-wiring-reflect

**Edit:**
- `loomcli/wiring_test.go`

**Create:**
- `loomcli/wiring_seams_test.go`

**Uses:**
- `internal/loomcli#loomCLI.wire`
- `testkit/envkit/envkit.go`

**Intent:**
Replace loomcli's hand-picked "seam is filled" tests with one reflective test (D8, #324 §5), so a new `shedrecipe.Env` seam is checked without a new test.

`wiring_seams_test.go` (untagged) holds `TestWire_EverySeamFilled`: it calls `wire()` on the same hermetic setup the existing wiring tests use, then requires `envkit.NilSeams(c.env)` to equal the `intentionallyNil` set exactly.
`intentionallyNil` maps each field path `wire()` legitimately leaves nil to a one-line reason, one entry per line without aligned values (D10); a nil field not listed fails, and a listed field that is no longer nil fails too, so the set cannot rot.

`NilSeams` sees only nil fields reachable from `shedrecipe.Env`, so it replaces exactly the assertions of that shape and nothing else:

- Deleted, because they only nil-check `Env` fields: `TestWire_WebsterRunIsFilled`, `TestWire_PlanSeamsFilled`, `TestWire_ReflectFrictionIsFilled` and `TestWire_SkipPlanReviewIsFilled`.
- Trimmed to their remaining assertions, dropping only the `Env` nil checks: `TestWire_WebsterDepsFullyPopulated` keeps its non-empty `Geom.WebsterDir` check and the `c.env.WebsterDeps.Geom == c.runDeps.Geom` identity check; `TestWire_DiscussionSeamsFilled` keeps `c.env.Shuttle == c.runner`; `TestWire_DescribeSeamsFilled` keeps its `DescriptionPath` value check; `TestWire_ReviewSegmentSeamsFilled` keeps its `StencilsDir` and `RunRoot` value checks.
  A trimmed test may be renamed to say what it still asserts.
- Kept whole, because their nil checks sit on fields outside `shedrecipe.Env`: `TestWire_LandingSeamFieldsPopulated` (`c.registry`, `c.runner` and the `c.landingCfg` value) and `wiring_commitstatus_test.go`'s `TestWire_CommitStatusFilled` (`c.shedPaths.CommitStatus`), so that file is not touched.

Tests asserting values, identity or a mode-specific shape also stay: `TestWire_PathFieldsMatchLoomengineAccessors`, `TestWire_RunLockDiffersFromStatusLock`, `TestWire_CwdIsToldToTheEnv`, `TestWire_RefMatcherIsRealScanner`, `TestWire_BisectorOpenerNonNilInHubOnlyMode`, `TestWire_ReviewTripleMatchesLoadedConfig`, the `*SpecEvaluatesToExpectedShape` tests, `TestWire_FrictionDirFillsBurlerAndWebster` and the `wireLightweight` tests.
`internal/loomcli/wiring_commitstatus_integration_test.go` is D9-protected and untouched.

Prove the test: add an unset func field to `shedrecipe.Env`, watch `TestWire_EverySeamFilled` fail naming it, then remove the field.

What the escape hatch lets through: a seam listed in `intentionallyNil` stays nil unchecked; each entry names one field path and its reason, and review owns additions.
List the deleted and renamed leaf tests in the commit body for the test-count accounting.

**ImpactSummary:** loomcli wiring tests only; four Env-only nil-check tests become one reflective test, four mixed tests lose only their Env nil checks, and every value, identity and non-Env assertion is kept.
