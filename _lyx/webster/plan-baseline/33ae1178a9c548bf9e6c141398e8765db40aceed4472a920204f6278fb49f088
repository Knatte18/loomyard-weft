# Card 28 — landingshed-shedfake

**Edit:**
- `landingshed/commitstatus_test.go`
- `landingshed/finalize_integration_test.go`
- `landingshed/finalize_test.go`
- `landingshed/finalize_verify_test.go`
- `landingshed/prgate_test.go`
- `landingshed/publish_integration_test.go`
- `landingshed/publish_test.go`
- `landingshed/publish_verify_test.go`
- `landingshed/verifygate_integration_test.go`

**Uses:**
- `testkit/shedfake/shedfake.go`

**Intent:**
Convert landingshed's tests to `shedfake` and leave the package exactly one local `Deps` builder (#324 §2, §3).

`envkit` will import `landingshed`, so landingshed's in-package tests cannot use it (D1): the package keeps one local builder.
Merge `newTestDeps` (`publish_test.go`) and `newFinalizeDeps` (`finalize_test.go`) into one `newTestDeps(t)`; each caller mutates the returned value for its own variant.
The integration-tagged `landingshed.Deps` literals in `finalize_integration_test.go`, `publish_integration_test.go` and `verifygate_integration_test.go` start from that builder and override the fields the real hub supplies.

`fakeResolutionShuttle` (`finalize_integration_test.go`) and any other no-op `mergeresolve.Shuttle` become `shedfake.MergeShuttle`; one that scripts a resolution sets `RunFn`.
`failingShuttle` (`publish_integration_test.go`) fails the test when called, which is behaviour the kit's fakes do not assert: it stays, or becomes a `MergeShuttle` whose `RunFn` calls `t.Fatalf` in the test.

Every `Call` site followed by the fatal-on-error and outcome-compare block becomes `shedfake.RequireOutcome`/`shedfake.CallOK`.
The log-capture helper in `publish_test.go` is card 38's.
Apply D10 to every touched test.

**ImpactSummary:** landingshed tests only; two local Deps builders become one, merge-shuttle fakes and Call boilerplate move to shedfake, assertions unchanged.
