---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/loomcli/start.go:167
    summary: Publish failure TestSmokeBootstrapLifecycle died_driver_proceeds_to_the_success_envelope_and_logs_why; currentProducer's strict status read refused lyx loom start with state decode failed before the driver ran, and the fix at HEAD has no tmux-tier run behind it
  - id: F2
    severity: BLOCKING
    class: design
    location: internal/loomcli/start.go:167
    summary: Publish failure TestSmokeBootstrapLifecycle malformed_status_proceeds_to_the_success_envelope_and_logs_why; same cause, a truncated status file refused the spawn with unexpected EOF, and the fix at HEAD has no tmux-tier run behind it
  - id: F3
    severity: BLOCKING
    class: design
    location: internal/loomcli/start.go:160
    summary: two refusals card 6 adds to lyx loom start and resume name no way forward and have no refusal-spec row, breaking PATTERN-refusal-way-forward
  - id: F4
    severity: LOW
    class: consistency
    location: internal/loomcli/start_midmerge_test.go:174
    summary: the branch leaves two files not gofmt-clean, start_midmerge_test.go (marker =shedrun) and internal/testkit/envkit/envkit.go (nilLegal keys misaligned)
  - id: F5
    severity: LOW
    class: design
    location: internal/loomcli/start_midmerge_test.go:324
    summary: the clean pair with a truncated status file row repeats the unknown-field row, since both reach the one state.ErrDecode branch of currentProducer
---

### [BLOCKING] Publish failure: died_driver_proceeds_to_the_success_envelope_and_logs_why

**Location:** `internal/loomcli/start.go:165-180` (`currentProducer`), called from `runDriverSpawnAndWait` at `internal/loomcli/start.go:317`.
**Issue:** The log (`.lyx/verify/publish-failure.log`) shows `state: decode failed: json: unknown field "__smoke_unknown_field__"` as the envelope of `lyx loom start`, and no `.lyx/loom/driver.log`, so the refusal came before any driver spawned.
The cause is the branch's own card 6, not the parent merge.
`currentProducer` read the status file with `state.ReadJSONStrict` before every spawn and returned any error as a refusal, so a status file the driver's own read gate is meant to diagnose now refused the bootstrap first.
`git diff 6235b4043^1 6235b4043` carries no change to the status schema, `state.ReadJSONStrict` callers or the driver-log writer; its only `internal/loomcli` hunks are smoke-test helpers (`runLoomCLIWithEnvNoFatal`, `FABRIC_SKIP_PUSH`).

The focus questions, settled by reading the code at HEAD:

- `currentProducer` is the only new strict status read on the `lyx loom start` path before the spawn.
  The other strict reads on that path are `runHaltedAtHandBack` (`start.go:115`, only for a live strand with no park marker, before no spawn), `vouchForSpawnedResume` (`start.go:225`, a Warn and return on error), the handshake's `halted` closure (`start.go:420`, a read error reports false) and `loomengine`'s Seed and `CheckSeedIncoherent` (which map `state.ErrDecode` to the seed-exists answer).
  None of these returns `ErrDecode` as a refusal.
  `resumeFromRunState` (`resume.go:54`) refuses on a decode error, as it did before the branch.
- Commit `d2ad9fc55` (after the log's run) makes `currentProducer` log a Warn and return `("", nil)` on `state.ErrDecode`.
  An empty producer is no landing row, so a parked mid-merge over an undecodable status file keeps the ordinary `merge_in_progress` refusal (`refuseOverUnfinishedMerge`, `start.go:197-209`), and a clean pair proceeds to the spawn, where the driver's read gate logs the decode failure.
  `start_midmerge_test.go` covers the decode case with the `own leftover under an undecodable status file` row of `TestRunDriverSpawnAndWait_MidMerge_SpawnRefusals` and the two undecodable rows of `TestRunDriverSpawnAndWait_MidMerge_CleanPairProceeds`, and the parked-merge case with the other rows of the first table.
- `lyx gate test ./internal/loomcli ./internal/landingshed ./internal/agentname ./contracts/stencils ./internal/testkit/envkit` passes at HEAD, and so does `lyx gate test --tags integration ./internal/loomcli ./internal/fabricengine ./internal/landingshed ./internal/shuttleengine ./cmd/lyx`.

Reading the code, the failure's cause is already removed at HEAD.
The failing test is `//go:build tmux`, a tier this reviewer cannot run, so no run of it at HEAD exists and the rubric requires the failure raised.
**Fix:** No production change found.
Run the tmux-tier `TestSmokeBootstrapLifecycle` at HEAD; if it is green, answer this finding with that run as the evidence the premise no longer holds.
If it still fails, the refusal's text names the remaining site.

### [BLOCKING] Publish failure: malformed_status_proceeds_to_the_success_envelope_and_logs_why

**Location:** `internal/loomcli/start.go:165-180` (`currentProducer`).
**Issue:** The same cause as the finding above, for the truncated-JSON shape: the log shows `state: decode failed: unexpected EOF` refusing the bootstrap with no driver log written.
`loomshed.Seed` already maps the decode failure to `ErrSeedExists` (`internal/loomshed/seed.go:64`), so the new `currentProducer` read was the only step that turned it into a refusal.
At HEAD the `ErrDecode` fallback covers it, and `truncatedStatusBody` is the unit-level row for it.
**Fix:** Settled by the same tmux-tier run as the finding above.

### [BLOCKING] Two new refusals name no way forward and have no refusal-spec row

**Location:** `internal/loomcli/start.go:153-163` (`readRecordedParentBranch`), `internal/loomcli/start.go:200-204` (its refusal), `internal/loomcli/start.go:317-321` (`currentProducer`'s non-decode error refusal).
**Issue:** `PATTERN-refusal-way-forward` binds loom: every refusal reachable through a loom verb names its way forward and has a row in `contracts/specs/refusal-spec.md`, landing with its test.
Card 6 adds two:

- `refuseOverUnfinishedMerge` refuses with the bare `err.Error()` when the origin record is absent or unreadable (`the pair has no fabric origin record`), for a parked merge-in at Publish or Finalize.
  The message names no verb and the spec has no row for it.
  The tests (`TestRunDriverSpawnAndWait_MidMerge_ProbeErrorRefuses`, `TestResumeVerb`) reach it only through the canned seam error `origin boom`, never the production message.
- `runDriverSpawnAndWait` refuses with the bare status-read error when `currentProducer` fails for a reason other than a decode error (a lock or I/O failure), also with no way forward and no row.

The pre-existing probe-error refusal has the same shape, but it is not new in this diff.
**Fix:** Give each message a way forward naming only allowed verbs (for the origin record, `lyx fabric reconcile` or `lyx loom start --parent <branch>` if that is what restores it; for the status read, retry the verb), add a `loom` row for each to `contracts/specs/refusal-spec.md` in the same change, and have a test row assert the production message, not a canned one.
Alternatively, fall back to the ordinary `merge_in_progress` refusal on an unreadable origin record, which already names its way forward and has its row.

### [LOW] Two changed files are not gofmt-clean

**Location:** `internal/loomcli/start_midmerge_test.go:174`, `internal/testkit/envkit/envkit.go:37-45`.
**Issue:** `gofmt -d` on both files reports a diff.
`start_midmerge_test.go` has `marker =shedrun.ParkMarker(...)` with the space missing, added by the `d2ad9fc55` fix.
`envkit.go`'s `nilLegal` map gained the key `"Landing.StopConflictSession"`, which is longer than the aligned column, so every value in the map is misaligned.
The other files `gofmt -l` lists (`internal/fabricengine/pull.go`, `internal/pattern/doc.go`, `internal/shell/posix.go`, `internal/shuttleengine/waiting_test.go`) are not touched by this branch.
**Fix:** Run `gofmt -w` on the two files.

### [LOW] A second undecodable-status row repeats the first

**Location:** `internal/loomcli/start_midmerge_test.go:324`.
**Issue:** The `clean pair with a truncated status file` row of `TestRunDriverSpawnAndWait_MidMerge_CleanPairProceeds` covers the same behavior as the `clean pair with a status file carrying an unknown field` row beside it: both reach the one `errors.Is(err, state.ErrDecode)` branch of `currentProducer`, and the test asserts nothing that depends on the decode shape.
The two shapes differ only in the smoke tier, where they are separate subprocess runs.
**Fix:** Drop the `truncatedStatusBody` row and constant, or keep it only with an assertion that tells it apart from the unknown-field row.

## Per-card mechanical check

- Card 1 (Edit groups): `MidMergeState` and `MidMerge` carry and fill `Verb`, `Source`, `SourceSHA` and `StartSHA` from `loadMergeState`; the integration test asserts them on the parked pair and empty on clean and foreign state; the package doc is updated.
  No finding.
- Card 2: `ReadConfigChanges` intersects the two diffs and returns `ParentTip`; the three planned rows and the `ParentTip` assertion are in `TestReadConfigChanges`; `configChangeNotice` names the parent tip and `TestFinalize_ConfigChangeNotice` asserts it; both package docs updated.
  No finding.
- Card 3: `recordFailure` reports whether it wrote, `verdict` takes `recorded`, the refusal-spec row, the orch role item and both claims are present, and `TestPublishVerify_FailureRecord` has the unwritable-record row.
  No finding.
- Card 4 (Rename and Create): `grep -rn conflictRole --include=*.go` finds no remaining use of the old name; `MatchesRole` is in `internal/agentname` and its assertions sit in `TestNumberRole` with no new top-level test.
  No finding.
- Card 5: `preMerge`, `newPreMerge`, `clear`, the three `Deps` seams, `conflictSessionStopper` and its wiring in `loomPreRun` and `buildLoomShed` match the card; the planned test cases are all in `TestPreMerge_ClearsOwnParkedMergeIn` and `TestConflictSessionStopper`; the budget rows rise by one each (`internal/landingshed` 67, `internal/loomcli` 183).
  No finding.
- Card 6: see the findings above; the seam, the guard, `TestNewLoomCLI_SetsInjectedSeams`, the `TestResumeVerb` rows, the spec row and the `docs/overview.md` lines are present.
- Card 7: `decisionDeps`, the caller check, its refusals, the spec row, the plan stencil change, the claims and the `docs/overview.md` line match the card.
  No finding.
- Card 8: only `TestAttach_UntrackedTerminalRecord_RespawnEligibleRegardlessOfAge` gained `runOutcomeStopped` in both sub-tables, as planned.
  No finding.

Test economy beyond the finding above: no new top-level test duplicates an existing one; `TestConflictSessionStopper` and `TestPreMerge_ClearsOwnParkedMergeIn` are the only new top-level tests and each is a table over its public surface.
