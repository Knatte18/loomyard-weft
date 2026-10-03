# Card 27 — shedadapters-bouncer-fixture

**Edit:**
- `shedadapters/bouncer_seed_test.go`
- `shedadapters/bouncer_clear_test.go`
- `shedadapters/bouncer_config_test.go`
- `shedadapters/bouncer_commit_test.go`
- `shedadapters/bouncer_judge_test.go`
- `shedadapters/bouncer_replay_test.go`
- `shedadapters/bouncer_skip_test.go`
- `shedadapters/bouncerprompt_test.go`
- `shedadapters/burler_test.go`
- `shedadapters/attachprobe_test.go`

**Create:**
- `shedadapters/bouncer_fixture_test.go`

**Uses:**
- `testkit/shedfake/shedfake.go`

**Intent:**
Replace shedadapters' parallel bouncer fixtures with one (#324 §6); this fixture serves one package, so it stays in that package's `_test.go` files (D1).

`bouncer_fixture_test.go` defines `newBouncerFixture(t, opts ...bouncerFixtureOpt)` returning the `BouncerConfig`, its stencils directory and, on request, the constructed `*Bouncer` over a `shedfake.Shuttle`.
Options cover what the copies vary: the rubric name and body, a specs-dir marker, a caller-supplied shuttle, and the burler-producer variants with or without an attach shuttle.

It replaces `testBouncerConfig`, `testBouncerConfigWithSpecsMarker`, `newTestBouncer` and `shippedBouncerStencilsFixture` (`bouncer_seed_test.go`), `newClearTestBouncerConfig` and `newClearTestBouncer` (`bouncer_clear_test.go`), `validBouncerConfig` and `newBouncerStencilsFixture` (`bouncer_config_test.go`), and `newTestBurlerProducer`/`newTestBurlerProducerWithAttach` (`burler_test.go`).
Each caller gets the same config values it had; where two copies built different values for the same field, the option the caller passes reproduces its own.

Apply D10 to every touched test.

**ImpactSummary:** Untagged shedadapters tests only; the parallel bouncer and burler-producer builders become one fixture with options, no assertion changes.
