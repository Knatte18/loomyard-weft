# Plan-Review round 1 — fixer report

Every finding was checked against the code and the decision record, and all seven are fixed.
`lyx loom validate-plan` reads `ok: true` after the fixes, with only informational `create-new-unit` findings.

## Fixed

- **F1 (BLOCKING, test budget).**
  Confirmed: `cmd/lyx/testdata/test-budget.yaml` pins `internal/shuttleengine` 143, `claudeengine` 91, `shuttlecli` 19, `orchengine` 122 and `battenshed` 89, and each of those packages already holds exactly that many top-level tests.
  Each card now says which planned tests are rows of an existing test and which are new top-level functions.
  Every card that adds a function lists `cmd/lyx/testdata/test-budget.yaml` under `**Edit:**` and raises the package's row by its own count:
  card 2 (+1 claudeengine), 3 (+1 claudeengine), 5 (+1 claudeengine), 6 (+1 shuttleengine), 7 (+2 claudeengine), 8 (+2 shuttleengine), 9 (+1 shuttlecli), 10 (+1 shuttleengine), 11 (+1 shuttleengine), 13 (+1 shuttleengine), 14 (+1 orchengine), 15 (+1 battenshed) and 17 (+1 shuttlecli).
  Tests that extend an existing test are now named as such, for example `TestReadWaitMarker`, `TestReadAgentActivity_ReadsLiveRunsAndTheWaitMarker`, `TestPollEventsTick_ShellExpiry` and `TestLoadConfig_TemplateDefaultsResolve`.
  `00-overview.md`'s Shared Decisions state the rule once.
- **F2 (BLOCKING, test economy).**
  Confirmed: `settings_test.go`'s `TestBuildDenyNotice_MatchesInstalledDenies` already asserts that the notice matches the installed denies over every combination.
  Card 1 drops its new row set and generalizes that test to read each installed entry back to its table row.
  Card 2 adds the python dimension to that test and its install rows to `TestBuildSettings_DenyToggleMatrix`, and keeps the `prepare_test.go` dimension.
- **F3 (MEDIUM, per-reader transcript-fallback turn end).**
  Card 3's `signals_test.go` table gains a row for this.
  A Stop line without `background_tasks` reads its shell under `SignalTranscript`.
  The same data parsed again, after the transcript gains the shell's completion notification, reads with no task.
  The row sits in card 3, not card 8, because the parse-time transcript read lives in `claudeengine`, and `sessionread_test.go` uses an engine double behind the provider seam.
- **F4 (MEDIUM, stale docs).**
  Card 4 adds `shuttleengine/doc.go`.
  It also rewords `activity.go`'s file comment and the `LastActivity` field comment to the stamp-based source with its mtime fallback.
  Card 10 adds `shuttleengine/doc.go`, where "two Go-side waits" becomes three with `held`.
  It also rewords `waitmark.go`'s file comment and the `WaitMarker` comment to name the third label and batten's skipping of it.
- **F5 (MEDIUM, `claudeengine/doc.go`).**
  Card 3 adds `claudeengine/doc.go` for the signal parse, the stamp pairing, the parse-time transcript fallback and the process-ending `SessionEnd` reasons.
  Card 7 adds it for registry liveness and when it reads unproven, the interrupt marker and the shared registry reader.
- **F6 (LOW, card 12 scope).**
  Card 12 is narrowed to the sentences card 11 makes false, which is D8's own consequence: D8 says done-at-once shells go into `ExpiredShells` "so webster's friction note still records them".
  The `Background shells waited out` heading is kept.
  `internal/webstercli/verbs_test.go` is dropped from the targets, because its comment describes a transcript-expiry scenario that stays true.
  The `doc.go` edit is limited to its "waited out after `background_shell_wait_min`" sentence.
- **F7 (LOW, output-files-first handover).**
  Card 6's `SessionFold` doc comment and card 8's `shuttleengine/doc.go` section now state the handover.
  `dead` and `unknown` outrank `idle-done`, so a finished run can read `dead`.
  The first consumer that acts on `dead` applies `PATTERN-completion-signal`'s output check first.

Cards 3 and 6 now carry `**Edit:**` targets, so each gained an `**ImpactSummary:**`.
The summaries of cards 4, 7, 10, 12 and 17 were updated to their new targets.

## Disputed

None.

## Deferred

None.
