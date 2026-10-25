```yaml
lens: affected-set
outcome: reported
```

Lens: affected-set. For each change a card makes, the set of readers and asserters I found, and which cards cover it.

**Per change: set found → covering cards**

- **`drift` config module registered in `configreg`** (card 1):
  - Members: `configreg.Modules`, `cmd/lyx` `configStrictnessStrictSet`, `docs/overview.md`, and `pattern/PATTERN-config-layers.md`'s "Strict callers" list.
  - Card 1 covers all but the PATTERN list. That list is already stale (darnengine and dobbyengine are missing from it).
- **Gate key `drift-verify`** (card 3):
  - Members: `internal/gateslot/template.yaml`, `tiers.go` `goReadGates`, `config_test.go` `templateDefaults`, `internal/gateslot/doc.go`, `envkit` `fixedGateLists`, `pattern/PATTERN-test-speed.md`, `docs/dotnet-hub.md`, and the example `gates` block in `internal/gatecli/doc.go:285-293`.
  - Card 3 covers all but the `gatecli/doc.go` example.
  - The migration fixtures at `config_test.go:534,558` are explicit documents, not the template, so they are unaffected.
- **`runproc` role `capture`** (card 5): `record.go` and `runproc/doc.go`. Tender renders roles generically. Card 5 covers it.
- **Breakaway limit in `createKillOnCloseJob`** (card 4): its callers are `StartTied`'s per-child job and `HoldNamedJob`, used by the loop and the capture. Card 4 covers both through the shared function.
- **New loom and darn recipe edges** (card 14):
  - Readers of those edges:
    - `loom-recipe.yaml` and `darn-recipe.yaml`, with their header comments.
    - `loomrecipe` `shape_test.go` `wantProducerTable`, `darnrecipe_test.go` (`wantDarnProducerTable`, `TestNewDarn_Routing`, `TestDarnRouting` lines 147/151/326) and `sequence_test.go` `deriveSequenceOrder`.
    - `internal/loomrecipe/progress_test.go:64`.
    - `internal/loomrecipe/doc.go:15,17` and `docs/overview.md:494-497,558-559`.
    - `interruptpolicy.go`, `reportengine` `loomPhases` and `darnPhases`, and `internal/reportengine/doc.go:129,131`.
  - Card 14 covers all but `progress_test.go:64` and `reportengine/doc.go:129,131`.
  - The fixture producer lists in `settledguard_test.go` and `finalizeguard_test.go` build their own recipes, so they are unaffected.
  - The escalation tests stub only named rows. The Drift rows reach them only as Skipped under the template's `drift.enabled: false`, which card 14 relies on.
- **Phase tables** (card 14): `timingspans.go` `loomPhases` and `darnPhases` are covered, but the prose table in `internal/reportengine/doc.go:125-131` is not.
- **Stencil registry** (card 8): `stencils.go` embeds, `entries`, `roleOpeningStencils`, `markers_test.go`, and the scans that walk the registry (`registry_test`, `parentdirective_test`, `roleskills_test`). Card 8 covers it.
- **`DecisionHaltPrefix` meaning widened** (card 11): its doc and `shedadapters/doc.go:112`. The readers in `arbiterstop.go:63,165` keep their behaviour. Card 11 covers it.
- **`lyx drift` subtree** (card 7): `newRoot`, plus the CLI-tree walk and registration tests, which cover it automatically. Card 7 covers it.
- **Unbounded wait file** (card 9): `unboundedWaitFiles` and `PATTERN-daemon-wakeups.md`. Card 9 covers it.
- **Teardown capture-kill step** (card 13): `Teardown`, `New`, `EndSession`, `fakeSubstrates.teardown`, `TestTeardown_CallSequence` and `pairteardown/doc.go`. Card 13 covers it.
- **Activity reading** (card 12): `agentActivityFor` and its test. Card 12 covers it.

**Findings**

1. **MEDIUM, design.** `14-drift-recipe-rows`: card 14 covers only part of the set that reads the loom edges it changes. It misses `internal/loomrecipe/progress_test.go:64`: `TestRouting_ProgressAt` pins Code-Bouncer's OnDone to Describe, so it fails under card 14's own gate once Code-Bouncer → Drift-Verify lands. It also misses the phase prose in `internal/reportengine/doc.go:129,131`, which lists each phase's rows and gains no Drift rows, although card 14 edits `loomPhases` and `darnPhases`.
   - Evidence: `internal/loomrecipe/progress_test.go:55-64`, `internal/reportengine/doc.go:129-131`.
   - Fix: add `internal/loomrecipe#TestRouting_ProgressAt` and `internal/reportengine/doc.go` to card 14's targets.
2. **LOW, consistency.** `03-drift-verify-gate-key`: card 3 updates `docs/dotnet-hub.md`'s example `gates` block but misses the same full-key example in `internal/gatecli/doc.go`'s package doc, which lists every gate key.
   - Evidence: `internal/gatecli/doc.go:285-293`.
   - Fix: add `internal/gatecli/doc.go` to card 3's Extends.
3. **NIT, consistency.** `01-drift-config`: card 1 adds `internal/driftengine` to `configStrictnessStrictSet` but leaves the "Strict callers" list in `pattern/PATTERN-config-layers.md` without it.
   - That list already omits darnengine and dobbyengine, so this widens existing drift.
   - Evidence: the "Strictness" section of `pattern/PATTERN-config-layers.md` and `cmd/lyx/configstrictness_test.go:63-73`.

Tool calls:
1. grep for every reader of the gate-key list (`code-burler-fix`) and for `runproc` role literals, to find the set beyond the pre-read.
2. grep for the gate-key mentions in the specs and docs that list keys, and for docs and tests naming the recipe edges and row order the new rows change.
3. Read `progress_test.go:40-75`, the `gatecli/doc.go` example block and `reportengine/doc.go:125-133`, to confirm the edge assertion, the full key list and the phase prose.
4. grep for tests and docs that pin the OnDone edges of Discussion-Bouncer, Plan-Bouncer, Code-Bouncer, Darn-Write and Preflight, to complete the edge set.
5. Read `escalation_test.go:112-140`, to check whether the darn escalation test's row switch is affected by the new rows.
