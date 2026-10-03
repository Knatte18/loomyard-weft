# Card 29 — producers-shedfake

**Edit:**
- `loomshed/batchifier_test.go`
- `loomshed/discussionwrite_test.go`
- `loomshed/gatefindings_test.go`
- `loomshed/loompreflight_test.go`
- `loomshed/planwrite_test.go`
- `loomshed/webster_test.go`
- `preflightshed/preflight_integration_test.go`
- `battenshed/create_test.go`
- `battenshed/innerrun_awaiting_test.go`
- `battenshed/innerrun_halted_test.go`
- `battenshed/innerrun_ide_test.go`
- `battenshed/innerrun_test.go`
- `battenshed/seamchild_test.go`
- `battenshed/teardown_test.go`
- `loomcli/smoke_attachprobe_test.go`
- `loomcli/parentreview_integration_test.go`
- `mergeresolve/mergeresolve_test.go`
- `frictionengine/reflect_test.go`

**Uses:**
- `testkit/shedfake/shedfake.go`

**Intent:**
Route the remaining producer packages' `Call` boilerplate through `shedfake` (#324 §2), so a `ShedProducer.Call` change touches the kit rather than each package.

Every `p.Call(context.Background())` followed by the standard fatal-on-error block, with or without the outcome compare, becomes `shedfake.RequireOutcome` or `shedfake.CallOK`.
A site asserting `Call`'s error, or folding outcome, pointer and error into one combined check with its own message (as several `battenshed` halted-state tests do), keeps its direct call: the helper only replaces the identical boilerplate.
Any local `Shuttle`, `BurlerRunner` or merge-shuttle fake in these files moves to the kit the same way as card 26.

`loomcli/parentreview_integration_test.go`'s `scriptedShuttle` becomes a `shedfake.Shuttle` whose `RunFn` keeps its per-spec script.
`loomcli/smoke_attachprobe_test.go`'s `refusingBurlerRunner` refuses on purpose and stays unless `shedfake.BurlerRunner`'s scripted error reproduces it exactly.

The two Run-only scripted shuttle fakes move to `shedfake.MergeShuttle`, since neither package is cycle-forced (shedfake imports neither, nor anything that does):

- `mergeresolve/mergeresolve_test.go`'s `fakeShuttle`: `results`/`errs` become `Results`/`Errs`, `runHook` becomes `DuringRun`, and assertions on `specs` read `Specs`; `newTestDeps` takes a `*shedfake.MergeShuttle`.
- `frictionengine/reflect_test.go`'s `fakeShuttle`, a copy of mergeresolve's without the hook, converts the same way.

Every scripted value and every spec assertion stays.
Apply D10 to every touched test.

**ImpactSummary:** Producer tests in loomshed, preflightshed, battenshed and one loomcli smoke file swap Call boilerplate for shedfake helpers, and mergeresolve's and frictionengine's in-package shuttle fakes become shedfake.MergeShuttle; no production code changes, assertions unchanged.

**Verify:** go test -tags smoke ./internal/loomcli/...
