---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    location: internal/loomcli/cli_test.go:75
    summary: Four added or changed comment lines end mid-phrase with no punctuation, breaking PATTERN-comment-line-breaks
  - id: F2
    severity: BLOCKING
    class: scope
    location: internal/orchengine/doc.go:233
    summary: Five added or changed comment lines keep two independent clauses on one line (comma plus and/so with its own subject and verb)
  - id: F3
    severity: BLOCKING
    class: scope
    location: internal/orchengine/watcher_test.go:489
    summary: Untagged tests the diff edits by more than one line run serially with no t.Parallel and no comment naming global state, breaking PATTERN-test-speed
---

Range: `status.json` gives `product.parent` `main`, and there is no `_lyx/loom/rework` directory, so the range is `git diff $(git merge-base main HEAD)..HEAD` (merge base `959025744`).
`go build ./...` and `go vet` over the touched packages are clean.
The review was the two focus sweeps, run mechanically over every added Go comment line (445 lines) and every untagged `_test.go` function the diff touches.
A changed line that is only the tail of a legacy fixed-column sentence, and that itself ends with punctuation, is not flagged: its earlier lines are untouched, and `PATTERN-comment-line-breaks` forbids rewrapping those.

### [BLOCKING] Comment lines that end mid-phrase

**Location:** `internal/loomcli/cli_test.go:75`, `internal/loomcli/entryobservation.go:62`, `internal/loomcli/smoke_test.go:640`, `internal/loomengine/seed.go:136`
**Issue:** Each is an added or changed line that ends without punctuation and continues on the next line:
- `cli_test.go:75` ends `accepts`, continued by `--no-attach, hidden: ...` on line 76 (both lines are new).
- `entryobservation.go:62` ends `whether it`, continued by the legacy line `matches the observed history length and state. The delete is ...`, which also holds two sentences.
- `smoke_test.go:640` ends `exactly as`, and line 641 holds the end of one sentence and the start of another.
- `seed.go:136` ends `for a condition the`.

The last three are one-phrase substitutions inside legacy fixed-column paragraphs, but the changed line is the mid-phrase one, so the sentence it sits in counts as changed.
**Fix:** Rewrite each affected sentence, and only that sentence, one sentence per line, breaking at the sentence end and at independent-clause boundaries.

### [BLOCKING] Two independent clauses on one comment line

**Location:** `internal/orchengine/doc.go:231`, `internal/orchengine/doc.go:233`, `internal/loomcli/integration_driverbootstrap_test.go:22`, `internal/shuttleengine/claudeengine/claudeengine.go:3`, `internal/orchcli/doc.go:27`
**Issue:** Each line joins two clauses that each have their own subject and verb:
- `doc.go:231`: `means turns ran without a reload, or a restarted watcher on an old cursor found an old boundary:`.
- `doc.go:233`: `... types nothing and leaves the baseline, so a later turn end evaluates the boundary again.`
- `integration_driverbootstrap_test.go:22`: `wire() itself spawns no process and resolves no cwd, so it carries none of that hazard,`.
- `claudeengine.go:3`: `The type carries only its submit settle, fixed at construction, and every method it implements is a pure function ...`.
- `orchcli/doc.go:27`: inside the parenthesis, `the driver removes itself after a hand-back, and an early restart races that removal and leaves no driver`.

**Fix:** Break each line before the conjunction, so every independent clause starts its own line.

### [BLOCKING] Serial untagged tests edited by more than one line

**Location:** `internal/orchengine/watcher_test.go:489` (`TestWatcher_HandoffCompleteClearsThenResumes`), `internal/orchengine/watcher_test.go:511` (`TestWatcher_ClearingTimeoutNeverTypesWhileBusy`), `internal/orchengine/watcher_test.go:1168` (`TestWatcher_ReloadTypesNothingWhenIdleProbeFails`), `internal/orchengine/watcher_test.go:1237` (`TestWatcher_AutoCompactionWaitsForIdleProbe`), `internal/shuttleengine/claudeengine/usage_test.go:137` (`TestCompactedSince_FindsNewestMainChainBoundaryAfterSince`), `internal/shuttleengine/claudeengine/session_test.go:124` (`TestCompactSessionSequence`, and its subtests), `internal/shuttleengine/claudeengine/session_test.go:153` (`TestClearSessionSequence`), `internal/shuttleengine/config_test.go:42` (`TestLoadConfig_TemplateDefaultsResolve`), `internal/shuttleengine/session_test.go:106` (`TestRunner_SessionMethods_ErrorOnPlainEngine`), `internal/shedadapters/attachprobe_test.go:50` (`TestBurlerProducer_AttachesToLiveRoundInsteadOfRespawning`)
**Issue:** Each test has more than one added or changed line in this diff, and none calls `t.Parallel()` as its first statement or carries a comment naming the process-global state that keeps it serial.
Tests beside them in the same files are parallel and share the same fixtures (`newWatchEnv`, `newFixture`, `New()`, `t.TempDir()`), so no global state is evident.
`TestWatcher_SkillSkipCauses` (logger output), `TestRun_Send_ConfirmsSubmission` (`inputSleep`), `TestRunVerb_RelativeProfileResolvesAgainstSeamCwd` (`t.Setenv`) and `TestStart_NeverAttaches` (`TMUX`) are serial with a comment naming the state, and are not findings.
**Fix:** Add `t.Parallel()` as the first statement of each test, and of each subtest with its own fixture, then run the packages with `-race` to confirm none depends on global state; where one does, add a comment naming that state instead.

## Focus departures

None.
