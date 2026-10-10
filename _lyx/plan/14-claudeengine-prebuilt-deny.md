# Card 14 — claudeengine-prebuilt-deny

**Edit:**
- `internal/shuttleengine/claudeengine#rawGoCommandPattern`
- `internal/shuttleengine/claudeengine#steerRawGoDeny`
- `internal/shuttleengine/claudeengine/settings_test.go`

**Uses:**
- `plan:internal/gateslot#PrebuiltLyxEnv`

**Prosa:**
- `pattern/PATTERN-no-denied-recovery.md`

**Intent:** D7's deny: a spawned agent's command that names the prebuilt-binary variable is denied, the way the slot and strand variables are today.
`rawGoCommandPattern` gains the variable's name in the alternative that matches the slot and strand variables, spelled from the same source those two are spelled from, and `steerRawGoDeny` says "the slot, strand and prebuilt-binary environment variables belong to lyx".
`settings_test.go`'s hook table gains a row with a Bash command naming the variable, expected denied, and the steer-text assertions follow the new wording.
`PATTERN-no-denied-recovery.md` gains the sentence that the raw `go` deny also refuses a command naming the slot, strand or prebuilt-binary variable, so no refusal, stencil or spec may spell one.

**ImpactSummary:** One regexp and one steer string change in the deny table; the two tests that assert them are the listed file.
