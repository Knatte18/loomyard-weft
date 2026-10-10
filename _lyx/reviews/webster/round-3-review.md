---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: internal/loomcli/start.go:174
    summary: readRecordedParentBranch repeats the body of the ParentBranch closure in wiring.go, and no test reaches the real reader
---

Range: no `_lyx/loom/rework/` round exists, so the range is `git diff $(git merge-base main HEAD)..HEAD` (parent `main`) plus the Publish merge-in diff `6235b4043^1..6235b4043`.
The merge-in is `main`'s own commits and carries no defect the branch owns; the Publish failure is judged below.

### [LOW] The recorded-parent-branch reader duplicates an existing closure and is never run by a test

**Location:** `internal/loomcli/start.go:174` (`readRecordedParentBranch`), `internal/loomcli/wiring.go:496`

**Issue:** `readRecordedParentBranch` is the same body as the `ParentBranch` closure in `wiring.go`: `fabricengine.ReadOrigin`, an error when the record is absent, then `origin.ParentBranch`.
Card 6 asks for "the same read `wire`'s `ParentBranch` closure makes", but two copies now decide the absent-record wording and behavior independently.
Every test substitutes the `recordedParentBranch` seam, and `TestNewLoomCLI_SetsInjectedSeams` only asserts it is non-nil, so the real reader's absent-record and read-error paths are not reached by any test (the `wiring.go` closure had none either).

**Fix:** have the `wiring.go` closure call `readRecordedParentBranch(location)`, so one function owns the read.
Add an absent-record row and a recorded-branch row for it to an existing table-driven test only if a fake-free origin fixture is cheap; otherwise leave it, since the failure path is a thin wrapper over `ReadOrigin`.

## Publish failure

Both named tests, `TestSmokeBootstrapLifecycle/died_driver_proceeds_to_the_success_envelope_and_logs_why` and `.../malformed_status_proceeds_to_the_success_envelope_and_logs_why`, are resolved at HEAD, so I raise no BLOCKING finding for them.

- Cause, from `.lyx/verify/publish-failure.log`: `lyx loom start` over a poisoned status file refused with `state: decode failed: ...` before spawning the driver, because `currentProducer`/`runHaltedAtHandBack` surfaced the strict read's `state.ErrDecode`.
  The smoke steps expect the success envelope and a driver log carrying the decode failure.
- Fix, commit `d2ad9fc55`: `readRunStatus` (`start.go:113`) maps `state.ErrDecode` to "not found" with a warn log, and `currentProducer` and `runHaltedAtHandBack` both read through it, so they agree that an undecodable status file never blocks `start`.
  Other read errors still refuse, naming a retry (`statusReadFailedMessage`).
- Evidence: `.lyx/verify/verified-tree.yaml` records the gate command for HEAD `b124b3554` (tree `5d3e2dde`), whose chain runs each of the two failing tests by name under `-tags tmux` and then the whole tmux tier, and `verify.log` shows all of it `ok`.
  The tmux tier is refused in this session, so I did not rerun it.
  `lyx gate test` over `internal/loomcli`, `internal/landingshed`, `internal/agentname` and `internal/fabricengine` passes at HEAD.
- The second smoke fix (`25e7df7cd`, `bootstrapUnlessSeeded`) lets either rigged step run when selected alone; it is test-only and adds no top-level test.

## Focus directive verdicts

1. Refusal-spec row for `start` reaching `refuseOverUnfinishedMerge`: the `start/resume: merge in progress` row (`contracts/specs/refusal-spec.md:262`) names both verbs and the Publish/Finalize exception, and its way forward ("resolve the merge as the message names, then re-run the verb") matches the message, which names the fabric remedy or git and the retry of the calling verb.
   Not a defect.
2. Bare `err.Error()` refusals in `start.go`:
   - the merge-state probe (`rerunMessage`, row `start/resume: merge state unreadable`, test `...MidMerge_ProbeErrorRefuses`),
   - the run-lock probe, the strand-table read, the retiring-strand removal and the park-marker removal (`rerunMessage`; rows `raw I/O` and `start: driver strand directory unreadable`; rows in `TestRunDriverSpawnAndWait_LLMArm_ReleasesLockOnFailure` assert `re-run "lyx loom start"`),
   - the status reads (`statusReadFailedMessage`, row `start: status read failed`, tests in `...LiveDriverWithoutParkMarker` and `...ProbeErrorRefuses`)
   all name a retry, have a row and have a test asserting the way forward.
   Not a defect.
   The `startLLMDriverArm`, `os.Executable`, driver-log open and `runCmd.Start` refusals later in `runDriverSpawnAndWait` still return a bare `err.Error()`, but they predate this branch and are outside the diff, so I did not flag them.
3. `currentProducer` and `runHaltedAtHandBack` agree: both read through `readRunStatus`, so an undecodable file is "no status" for each and never blocks `start`.
   The rule is covered by `undecodable status file is a no-op` (`TestRunDriverSpawnAndWait_LiveDriverWithoutParkMarker`), `clean pair with a status file carrying an unknown field` (`..._CleanPairProceeds`) and `own leftover under an undecodable status file` (`..._SpawnRefusals`, which pins that the merge refusal then stands because the producer is unknown).
   Not a defect.

## Other checks

- Card 6's contract holds: `ownMergeInLeftover`, `currentProducer`, the re-signed `refuseOverUnfinishedMerge`, the `recordedParentBranch` seam set in `newLoomCLI` and asserted in `TestNewLoomCLI_SetsInjectedSeams` with its `testtiming:keep` line naming the field, both resume probe sites passing `status.CurrentProducer`, the help texts, the spec row and the `docs/overview.md` line; the guard reads the parent branch only for a parked `merge-in` at Publish or Finalize, and tests pin that with a failing seam.
- `PATTERN-shuttle-stop`: `conflictSessionStopper` stops through `c.runner.StopStrand`, shuttle's stop verb.
- `PATTERN-refusal-way-forward` and `PATTERN-no-denied-recovery`: the new landing Stuck reasons and `decision add` refusals each carry a way forward naming only allowed verbs, with spec rows in the same commit.
- Test budget: `loomcli` and `landingshed` rows were each raised by one, matching the one new top-level test each (`TestConflictSessionStopper`, `TestPreMerge_ClearsOwnParkedMergeIn`); the other new cases are rows in existing tables.
