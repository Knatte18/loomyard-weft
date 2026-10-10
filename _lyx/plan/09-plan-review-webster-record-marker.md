# Card 9 — plan-review-webster-record-marker

**Edit:**
- `internal/shedadapters#ReadRubric` -> `func ReadRubric(stencilsDir, name, specsDir, publishFailure, websterRecord string) (string, error)`
- `internal/shedadapters#Bouncer.runSeedSpawn`
- `internal/shedadapters#Bouncer.judgeCall`
- `internal/shedadapters#BouncerConfig`
- `internal/shedadapters/rubric_test.go`
- `internal/shedrecipe#burlerRoundProfile` -> `func burlerRoundProfile(cfg Config, stencilsDir, specsDir, publishFailure, websterRecord string) (burlerengine.Profile, error)`
- `internal/shedrecipe#burlerRoundEntry`
- `internal/shedrecipe#bouncerEntry`
- `internal/shedrecipe#Env`
- `internal/shedrecipe/entries_bouncer_test.go`
- `internal/loomcli#loomCLI.wire`
- `internal/loomcli/wiring_test.go`
- `//contracts/stencils/loom/loom-rubric-plan-review.md`
- `//contracts/stencils/rubric_test.go`

**Create:**
- `plan:internal/shedadapters#Bouncer.websterRecordNote` -> `func (b *Bouncer) websterRecordNote() string`

**Uses:**
- `plan:internal/loomshed#WebsterRecordNote`

**Intent:** Carry webster's run record into the Plan-Review rubric through an optional `webster_record` marker, filled by Go the way `publish_failure` is (D2).
`ReadRubric` takes the note and fills the marker, `none` for a blank note, and a rubric without the marker ignores it.
`BouncerConfig` gains `WebsterRecord func() string`, nil rendering `none`; `websterRecordNote` reads it as `publishFailureNote` reads its sibling, and the seed and judge calls pass both notes.
`Env` gains `WebsterRecord func() string`; `bouncerEntry` hands it to the config, and `burlerRoundEntry` reads it once and passes it to `burlerRoundProfile`, so the Plan-Burler fixer sees the record in the rubric its explore step already carries.
`loomCLI.wire` sets it to a closure over `loomshed.WebsterRecordNote(anchorPath)`, called each time a rubric is read.
The rubric stencil gains the marker under a short heading stating the write surface; it points at the record and duplicates no contract (PATTERN-producer-pointer-rule).
`rubric_test.go` in contracts adds `webster_record` to the marker allowlist and pins that only the Plan-Review rubric renders it, as it pins `publish_failure` to the Webster-Review rubric; `TestReadRubric` gains rows for a blank and a filled note; `TestBouncerEntry_HappyPath` asserts the config carries the env's closure; `TestWire_ReviewKeysReachTheEnv` asserts the env's `WebsterRecord` is set.

**ImpactSummary:** ReadRubric's four callers (two Bouncer methods, the burler profile helper and its test) and the recipe env change in step; every other bouncer and burler row renders exactly as before since their rubrics carry no such marker.
