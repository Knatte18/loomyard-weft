# Card 34 — loomcli-wiring-reflect

**Edit:**
- `loomcli/wiring_test.go`
- `loomcli/wiring_commitstatus_test.go`

**Create:**
- `loomcli/wiring_seams_test.go`

**Uses:**
- `internal/loomcli#loomCLI.wire`
- `testkit/envkit/envkit.go`

**Intent:**
Replace loomcli's hand-picked "seam is filled" tests with one reflective test (D8, #324 §5), so a new `shedrecipe.Env` seam is checked without a new test.

`wiring_seams_test.go` (untagged) holds `TestWire_EverySeamFilled`: it calls `wire()` on the same hermetic setup the existing wiring tests use, then requires `envkit.NilSeams(c.env)` to equal the `intentionallyNil` set exactly.
`intentionallyNil` maps each field path `wire()` legitimately leaves nil to a one-line reason, one entry per line without aligned values (D10); a nil field not listed fails, and a listed field that is no longer nil fails too, so the set cannot rot.

It replaces the tests that only nil-check fields `wire()` fills: `TestWire_WebsterRunIsFilled`, `TestWire_WebsterDepsFullyPopulated`, `TestWire_LandingSeamFieldsPopulated`, `TestWire_DiscussionSeamsFilled`, `TestWire_DescribeSeamsFilled`, `TestWire_PlanSeamsFilled`, `TestWire_ReviewSegmentSeamsFilled`, `TestWire_ReflectFrictionIsFilled` and `wiring_commitstatus_test.go`'s `TestWire_CommitStatusFilled`, plus `TestWire_SkipPlanReviewIsFilled` if it only nil-checks.
Tests asserting values, identity or a mode-specific shape stay: `TestWire_PathFieldsMatchLoomengineAccessors`, `TestWire_RunLockDiffersFromStatusLock`, `TestWire_CwdIsToldToTheEnv`, `TestWire_RefMatcherIsRealScanner`, `TestWire_BisectorOpenerNonNilInHubOnlyMode`, `TestWire_ReviewTripleMatchesLoadedConfig`, the `*SpecEvaluatesToExpectedShape` tests, `TestWire_FrictionDirFillsBurlerAndWebster` and the `wireLightweight` tests.
`internal/loomcli/wiring_commitstatus_integration_test.go` is D9-protected and untouched.

Prove the test: add an unset func field to `shedrecipe.Env`, watch `TestWire_EverySeamFilled` fail naming it, then remove the field.

What the escape hatch lets through: a seam listed in `intentionallyNil` stays nil unchecked; each entry names one field path and its reason, and review owns additions.
List the deleted leaf tests in the commit body for the test-count accounting.

**ImpactSummary:** loomcli wiring tests only; the hand-picked nil-check tests become one reflective test, value and identity tests are kept.
