---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: tools/tokencount/count.go:581
    summary: Calibration takes the Webster engine from the first located webster-role session in engine-name order, while the merged live base uses the engine Master's run record names
    fix: In tools/tokencount/count.go where RunTally.WebsterEngine is set, keep the first-located engine as the fallback but prefer the masterEngine of the run's recorded websterengine.State, which tools/tokencount/profile.go readRecordedState already reads; when the record names none, keep today's behavior. Alternatively, state in the RunTally.WebsterEngine comment and in main.go's calibration paragraph that the derivation assumes every webster-role session of a run ran under one engine.
---

### [LOW] Calibration's Webster engine can differ from the engine the live Merriam base counts

**Location:** `tools/tokencount/count.go:581`, `tools/tokencount/calibrate.go:318`

**Issue:** The parent changed the live `websterengine.MerriamBase` to take the instruction files of Master's engine, through `MasterInstructionFiles(engines, roles[RoleMaster])` and a `State.MasterEngine` recorded beside `MasterSessionID`.
The task's calibration reconstructs the same base from `websterInstructionFiles`, which reads `reads[run.WebsterEngine].Instructions`.
`WebsterEngine` is the engine that located the run's first counted webster-role session, in `locateSessions` order, which is sorted by engine name.
While one engine is registered the two agree, and decision D8 and its parallel-task note ask only that they list the same files.
With a second engine whose worktree holds webster-role sessions, the located-first engine can differ from the engine Master ran under, and calibration would then price a different instruction-file set than the live base.
The parent's records now carry `masterEngine`, so the faithful source exists and the task's `profile.go` already reads the same state.
This is a latent mismatch, not a present defect, and it breaks no pattern.

## Checked and found sound

- No task code builds a `Spec` or `OpenSpec` literal with `Engine` set outside the promoted `ModelChoice`, and none reads `Result.Engine`; the build verify covers the compile side.
- `tools/tokencount`, `claudeengine`, `shuttleengine/transcriptread.go` and `shuttleengines` read no `modelspec.Resolved` field and resolve no model key, so `PATTERN-model-choice-mapping` holds for the task's code.
- The task declares no requirements and wraps no refusal, so the parent's `ErrRequirementsUnmet` and `ModelKey.Wrap` contract is untouched.
- `shuttleengines.RecordReaders` builds engines through the same registry as `SessionNamers`, the task's bullet in `PATTERN-shuttle-provider-seam.md` survived the merge beside the parent's one-line trim in `PATTERN.md`, and `internal/shuttleengine/doc.go` reads coherently with both sides' paragraphs.
- Tokencount calls `websterengine.MerriamBaseOf`, `PartitionBatch`, `State` and `BatchState`, all unchanged by the parent except for additive fields; the parent's `MerriamBase(geom, instructionFiles)` signature is not called by tokencount.
- Test budget: the merged rows for `internal/shuttleengine` (178), `claudeengine` (113), `internal/shuttlecli` (27) and `internal/shuttleengines` (6) equal the top-level tests each package holds, `TestMain` excluded, by the budget test's own counting rule.

## Rejected

- The `PATTERN.md` one-line summaries the parent shortened (`PATTERN-shuttle-provider-seam`, `PATTERN-pair-teardown`, `PATTERN-shed-driver`, `PATTERN-guarded-interrupt`) are the parent's edit alone, and the task's code upholds the longer background text too.
- No test the merge made redundant was found: the parent's `enginedispatch_test.go` and `requires_test.go` additions cover dispatch and refusals, which no task test pins.
