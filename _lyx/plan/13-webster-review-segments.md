# Card 13 — Webster and review specs name their segment

**Edit:**
- `internal/websterengine/roles.go`
- `internal/websterengine/runlevel.go`
- `internal/websterengine/runlevel_test.go`
- `internal/websterengine/recoverbatch.go`
- `internal/websterengine/recoverbatch_test.go`
- `internal/burlerengine/engine.go`
- `internal/burlerengine/engine_test.go`
- `internal/shedadapters/bouncer.go`
- `internal/shedadapters/bouncer_judge_test.go`
- `internal/shedadapters/bouncer_seed_test.go`
- `internal/treadleengine/judge.go`
- `internal/treadleengine/targeting.go`
- `internal/treadleengine/judge_test.go`

**Uses:**
- `internal/segmentcolor/segmentcolor.go`
- `internal/shuttleengine/spec.go`

**Intent:**
The remaining spawning modules name their role's segment on their launch specs, completing D13's rule that every spawning module's launch spec names one.

Segments, per D13's table:

- Webster's Merriam spec (`runlevel.go`, role `MerriamStrandRole`) and its recovery spec (`recoverbatch.go`): `segmentcolor.Webster`.
- Burler's reviewer and fixer specs (`burlerengine/engine.go`): `segmentcolor.Review`.
- Bouncer's judge and seed specs, at all four `shuttleengine.Spec` literals in `bouncer.go`: `segmentcolor.Review`.
- Treadle's judge and targeting specs (`judge.go`, `targeting.go`): `segmentcolor.Review`, since both are judges of a review loop.

Where a role constant is declared, its comment block gains the segment beside it, so a new spawn site in that module finds both; Webster's `MerriamStrandRole` and `RoleRecovery` are declared in `roles.go`.

Tests: an existing test per spec builder asserts the built spec's `Segment`, adding no test function, so no package's test-budget row changes:

- Merriam: `runlevel_test.go`, beside the `Role` it already asserts; burler: `engine_test.go`'s fake shuttle, beside the `Role` it already keys on.
- Recovery: `recoverbatch_test.go`'s `TestRecoverBatch_FirstCallSpawnsArchivesStaleReportAndStopsLiveStrand` asserts the `Segment` on the `reedengine.AddSpec` its real `shuttleengine.Runner` handed `shuttlefake.Reed` (`AddedSpecs`), which card 11 fills from the spec, since no recovery test inspects the built spec itself.
- Bouncer: `bouncer_judge_test.go`'s `TestBouncer_PromptComposition`, which already spawns both the seed and the judge spec and reads each through `shedfake.Shuttle.GotSpec`, asserts the `Segment` beside its `Skills` check; `bouncer_seed_test.go` asserts it on the seed spec it already captures.
- Treadle: `judge_test.go`'s `TestSpawnedRoles_SkillAndParentDirective`, whose table already runs the circling judge, the milestone judge and the targeting call against one fake shuttle, asserts the `Segment` beside its `Skills` check, covering `judge.go` and `targeting.go` alike.

**ImpactSummary:** spec literals in the webster, burler, shedadapters and treadle packages gain one field each; their runs get a color, nothing else changes.
