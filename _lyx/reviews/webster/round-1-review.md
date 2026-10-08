---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: internal/shuttleengine/claudeengine/usage_test.go:255
    summary: New tests run serially without a comment naming process-global state, against PATTERN-test-speed (TestSessionUsage_SumsParentAndForks, TestBurlerProducer_Call_WritesRoundUsageRecord, TestWriteRoundFacts_LensSection, TestBurlerProducer_Call_ClusterExclusions)
  - id: F2
    severity: LOW
    class: design
    location: internal/loomengine/config.go:432
    summary: validateFanKeys reads burler.yaml on every LoadConfig even when discussion_fan and plan_fan are both empty, so an unset fan no longer behaves as before
  - id: F3
    severity: LOW
    class: consistency
    location: internal/shuttleengine/claudeengine/usage.go:311
    summary: transcriptTokens has no caller in production or tests, and the sumTranscriptTokens doc comment is written in terms of it
  - id: F4
    severity: LOW
    class: consistency
    location: internal/loomengine/config.go:321
    summary: Files the diff touches are not gofmt-clean (config.go, config_test.go, claudeengine/usage_test.go)
---

Range: `_lyx/shed/review-lens-fan/status.json` gives `product.parent` as `main`, and `_lyx/loom/rework/` does not exist, so the target is `git diff $(git merge-base main HEAD)..HEAD`, which is `0ebdea27f..HEAD`.
`go build ./...`, `go vet ./...`, `go vet -tags integration ./...` and `go test ./...` all pass on the target.

Every card's Create and Edit members exist with the stated signatures (`applyClusterExclude`, `writeRoundFacts` match their re-sign arrows), and no card has a `Rename` or `Delete` group, so no per-type mechanical check was owed.
The focus questions on cards 1 to 14 came out as the plan states, except for the points below:

- **Cards 7 to 9**: `excludedLenses` reads a round's eligibility only from `round-<k-1>-usage.yaml`, and a missing or unreadable record yields an empty `ran` set, so entries become ineligible and no lens is dropped from the fan.
  The facts file reads the lenses and fan from the same record.
- **Cards 8 and 10**: exclusions are unioned over focus files 2 to N in `runDir` only (one generation per run directory), round 1 drops them, and only the judge call is passed `focusSchemaMarkers(b.cfg.ClusterExcludes)`; the seed call passes `false`.
- **Cards 1 and 3**: the held fallback applies when a round's exclusion would empty the fan, and fan and lens names resolve per name over the embedded template.
- **Cards 4 to 6**: `SessionUsage` and `UsageReader` live in `shuttleengine` with no Claude shape knowledge; the Claude reader sums the parent and every fork transcript; `Half` and `Result` carry usage, times and lenses.
- **Cards 11 to 13**: an env fan reaches both rows of a segment through `RowClusterFans`, Webster rows are absent from the map, and a nil map leaves every row building as before.
- **Card 14**: the guard rule function, its tests and the Webster-Burler recipe comment match the plan and the recipe's rows.

### [BLOCKING] New tests omit t.Parallel with no comment naming global state

**Location:** `internal/shuttleengine/claudeengine/usage_test.go:255`, `internal/shedadapters/burler_test.go:492`, `internal/shedadapters/burler_test.go:681`, `internal/shedadapters/bouncerfacts_test.go:271`
**Issue:** `PATTERN-test-speed` requires `t.Parallel` unless the test touches process-global state, and a comment at the test or atop its file must name that state.
`TestSessionUsage_SumsParentAndForks` calls `t.Setenv("HOME", ...)` and `TestBurlerProducer_Call_ClusterExclusions` uses `logcapture.Capture`, and neither has a comment naming the global state.
`TestBurlerProducer_Call_WritesRoundUsageRecord` and `TestWriteRoundFacts_LensSection` touch only `t.TempDir` fixtures and fakes, yet neither calls `t.Parallel`.
Sibling tests in these files also omit it, but a sibling's omission does not license a new one under the repo's constraint.
**Fix:** Add `t.Parallel()` to the two tests that touch no global state (and their subtests where independent), and add a one-line comment naming `HOME` and the global logger on the two that do.

### [LOW] validateFanKeys reads burler.yaml although no fan key is set

**Location:** `internal/loomengine/config.go:432`
**Issue:** Card 12 has `LoadConfig` resolve "each non-empty fan key", and the plan's impact line says nothing changes when the fans are empty.
`validateFanKeys` calls `burlerengine.LoadConfig(baseDir)` first, unconditionally, so every `loomengine.LoadConfig` caller (including the arm path in `internal/loomcli/arm.go`) now fails on a malformed or unreadable `burler.yaml` with both fan keys empty, which did not happen before.
**Fix:** Return nil at the top of `validateFanKeys` when `cfg.DiscussionFan` and `cfg.PlanFan` are both empty, and load `burler.yaml` only when one is set.

### [LOW] transcriptTokens is dead code and the next doc comment depends on it

**Location:** `internal/shuttleengine/claudeengine/usage.go:311`
**Issue:** `SessionUsage` calls `sumTranscriptTokens` directly, and a grep over the repository finds no caller of `transcriptTokens` in production code or tests, so the wrapper only exists because card 5 names it.
`sumTranscriptTokens`'s doc comment then reads "is transcriptTokens that also counts sidechain entries", which defines the live function in terms of the dead one and breaks the self-contained-comment rule.
**Fix:** Delete `transcriptTokens` and reword the `sumTranscriptTokens` doc comment to state what it sums on its own (or make `SessionUsage` the caller of `transcriptTokens` for the parent transcript, if the card's member is to stay).

### [LOW] Touched Go files are not gofmt-clean

**Location:** `internal/loomengine/config.go:321`, `internal/loomengine/config_test.go:89`, `internal/loomengine/config_test.go:288`, `internal/shuttleengine/claudeengine/usage_test.go:262`
**Issue:** `gofmt -l` lists all three files and none was listed at the merge base.
The `Judge` field in `Config` and in `templateConfig()` lost its column alignment after the three new fields were inserted above it, the table row `{"unknown fix_start","fix_start", ...}` lost a space, and the `tests` struct in `TestSessionUsage_SumsParentAndForks` has misaligned fields.
**Fix:** Run `gofmt -w` on the three files.
