```yaml
lens: forced-edits
outcome: reported
```

Forked lens review, forced-edits lens: I checked what each card's changes force outside the members the card names.

1. **MEDIUM — Card 11 (D7): a failure-report test the triage lines change is not listed.**
   - `internal/gatecli#TestGateTest_Scenario` (`internal/gatecli/test_integration_test.go:673`) runs a hub with `"scoper": "go"` (`:635`).
   - Its subtest at `:1415` makes a real go test tier fail `TestFails` and asserts the failure report around `:1442`.
   - Card 11 builds the triage lookup in any hub call with scoper `go` in which a run failed a test. A test no map holds gets "not in the coverage maps" as its first four-space line, so this report gains a line.
   - Card 11 lists only `TestRenderFailureReport` and `TestFailedTests`. `TestGateTest_Scenario` belongs on the card, so its report assertion is checked against the new line.

2. **MEDIUM — Card 9 (D3 wiring): the envkit seam list misses the weaver geometry's new field.**
   - Card 9 adds `weaverengine.Geometry.ReachPending`, nil turning the turn off. envkit walks `Env.WeaverDeps.Geom.*`: `nilLegal` holds `"WeaverDeps.Geom.Git"`, `"WeaverDeps.Geom.GateSlots"` and `"WeaverDeps.Geom.GateConfig"` (`internal/testkit/envkit/envkit.go:56`).
   - The Intent adds only `VerifyReach` to `nilLegal`. Without `"WeaverDeps.Geom.ReachPending"`, `TestNilSeams_*` and `TestFullEnv_FillsEverySeam` report the new nil seam.
   - Name the entry in the Intent; the same holds for any other geometry the walk enters.

3. **LOW — Card 6 (D3): existing "built only on command" text becomes false, and no card names it.**
   - Card 6 has the reach step refresh worktree maps through `testMapRun`. These lines say no gate builds maps:
     - `internal/gatecli/doc.go:1030`: "It runs only on command: no dobby, landing or gate starts it".
     - `internal/gatecli/doc.go:5` and the file comment at `internal/gatecli/testmap.go:2`: "on command".
     - `docs/overview.md:842`: "No dobby builds the coverage maps: the operator builds them on command".
     - `docs/overview.md:974`: "only on command".
   - Card 6 lists `internal/gatecli/doc.go` only under `**Extends:**`, which adds a section. Its overview change names only the gate passage.
   - Move `internal/gatecli/doc.go` to `**Edit:**` and name these sentences, plus `internal/gatecli#testMapRun`'s file comment.

4. **LOW — Cards 5, 6 and 14: lists of the runs that share the redundancy bounds and the memory guard are not updated.**
   - The reach refresh (card 6) and `lym gate mutate` (card 14) run under these settings, but these passages enumerate only the existing runs:
     - the `template.yaml` comments on `memory_quit_mb`, `redundancy_build_timeout_min`, `redundancy_test_timeout_sec` and `redundancy_test_memory_mb` (`internal/gateslot/template.yaml:3,5-7`);
     - the memory-guard sentence at `internal/gatecli/doc.go:725`;
     - "Three redundancy runs take a tier's env and a gate slot" at `pattern/PATTERN-test-speed.md:220`.
   - Card 5 edits `template.yaml` only to add keys. Card 14 adds `lym gate mutate` to the PATTERN-test-speed list, but no card adds the reach refresh, so the count "Three" is wrong after cards 6 and 14.

5. **LOW — Card 13 (D9): `fileSites` leaves a signature question open, with no re-sign arrow.**
   - `internal/testcover#fileSites` is under `**Edit:**` with "`fileSites` may gain what the planner needs to tell a call into `internal/logger` from other calls".
   - If its parameters or results change, the bullet needs the re-sign arrow, and its caller `PlanMutants` must be listed.
   - As written, this is a research step. State the change: a tag on the existing site value with no signature change, or the new head.

Tool calls:
- One grep batch: the package clauses of the test files in the touched packages, consumers of the report name and the redundancy keys, `WeaverGeometry` and `nilLegal` sites, and gate verb lists. My context did not have the test package clauses or which callers build the weaver geometry.
- A second grep batch: import edges between shuttleengine, verifytree, gatecli and shedrecipe; `WeaverGeometry` callers; gatecli tests that assert report text; reflective `NumField` tests; the template's comments that enumerate runs; and "on command" text. My context did not have the import cycle risk, the reflective walks, or the docs that enumerate runs.
- `lym quarry resolve` on every existing glyph the cards list. A bad filter hid the statuses, so this result was unusable.
- `lym quarry resolve` again over the same glyphs, filtering for not-found statuses. Nothing printed, but I could not yet tell whether the filter worked.
- `lym quarry resolve` on the doubtful glyphs with a known-bad control, and a grep locating `TestRenderVerifyGateFindings`. This confirmed every listed glyph is found, so the previous empty result was real.
- A grep of `test_integration_test.go`'s report assertions and a read of `internal/weavercli/wiring.go:15-45`. My context did not show whether `TestGateTest_Scenario` asserts the failure report, or whether weavercli runs batches; it builds the geometry only for reset, so it needs no wiring.
