# Card 59 — plan-edit-guard

**Edit:**
- `internal/websterengine/fingerprint.go`
- `internal/websterengine/beginbatch.go`
- `internal/websterengine/recordbatch.go`
- `internal/websterengine/recoverbatch.go`
- `internal/websterengine/doc.go`
- `internal/webstercli/validate.go`
- `internal/webstercli/recordbatch.go`
- `internal/webstercli/recoverbatch.go`
- `internal/websterengine/fingerprint_test.go`
- `internal/websterengine/recordbatch_test.go`
- `internal/websterengine/recoverbatch_test.go`
- `internal/webstercli/verbs_test.go`
- `internal/websterengine/template_test.go`

**Prosa:**
- `contracts/stencils/webster/webster-template-master.md`

**Uses:**
- `internal/websterengine/rebaseline.go`
- `internal/websterengine/cardverify.go`
- `internal/websterengine/planbaseline.go`

**Intent:**
Fix PR #321 round-4 finding D's first item.
`lyx webster validate` and `record-batch` re-record the plan hashes without checking what changed.
Editing `00-overview.md` and running `validate` skips every `rebaseline` refusal, and a fork that weakens its own card's Verify through Bash, which no Write/Edit audit sees, has the weakened Verify re-run by `rerunCardVerifies` and baked into `PlanFileHashes` by the restamp.
Each restamp in these verbs exists to adopt webster's own rewrites (handle canonicalization, binding, the exact-tier drift repair), so the check belongs before the verb's first rewrite, where any difference is someone else's edit.

In `fingerprint.go`:

- `PlanEditError(st *State, planDir string) error` returns nil when the on-disk plan's fingerprint equals `st.PlanFingerprint`.
  Otherwise it returns the `ErrFingerprintMismatch` wrap `BeginBatch` builds today, with `fingerprintMismatchWayForward`.
  `BeginBatch`'s fingerprint check calls it, so the two texts cannot drift apart.
- `batchCardEditError(bs *BatchState, b batcher.Batch, planDir string) error` compares `batchCardHashes(b, planDir)` with `bs.CardHashes` when the record carries them, and returns an `ErrFingerprintMismatch` wrap naming each `batch NN card <id> changed since it was begun` with `fingerprintMismatchWayForward`.
- `fingerprintMismatchWayForward` adds, in every branch, `or restore the plan the run recorded with "lyx webster restore-plan"`.

In `recordbatch.go`'s `RecordBatch`, after the correctness-findings branch and before attribution advances, `PlanEditError` and then `batchCardEditError` run against `deps.Geom.PlanDir`.
A refusal there mutates nothing, so the batch stays open, the transcripts stay unseen, and the card verifies never run on the edited plan.
A fork plan write the audit sees still fails the batch first through its correctness finding.

In `recoverbatch.go`'s `PersistRecoveryTerminal`, the same two checks run after `checkRecoveredSuspects` and before `postBatchChecks`.

In `internal/webstercli/validate.go`, with a run in progress, `PlanEditError` is evaluated before `scopedValidate`.
On a mismatch the verb still lints, but it skips `RestampPlanBaseline`, leaves `state.json` untouched, and exits non-zero with the mismatch text as its message, carrying the findings as its blocking-findings envelope does.
`Long` says so, replacing its claim that validate re-baselines after any rewrite.

`internal/webstercli/recordbatch.go` and `internal/webstercli/recoverbatch.go` emit `plan_drifted: true` with the error when it wraps `websterengine.ErrFingerprintMismatch`, as `begin-batch` does for `ErrPlanDrifted`.

The Master stencil's "A plan-drift refusal ends your run as stuck" section covers `record-batch` and `recover-batch` refusing with `{"plan_drifted": true}` too, with the same rung: write `outcome: stuck` quoting the refusal, and do not retry.
Its operator sentence adds `lyx webster restore-plan` beside `rebaseline --card`.
Keep semantic line breaks and change nothing else in the stencil.

`doc.go`'s re-baseline paragraph states that validate, record-batch and recovery refuse a plan that changed before their own rewrites.

Tests:

- `fingerprint_test.go`: `PlanEditError` is nil on an unchanged plan and wraps `ErrFingerprintMismatch` naming `rebaseline --card` and `restore-plan` after a card edit;
  `batchCardEditError` names the edited begun card.
- `recordbatch_test.go`: `TestRecordBatch_RefusesPlanEditedSinceBegin`: a card's Verify weakened on disk after begin-batch, with an OK report → `ErrFingerprintMismatch`, the batch non-terminal, `SeenForkTranscripts` and `PlanFileHashes` unchanged.
- `recoverbatch_test.go`: `TestPersistRecoveryTerminal_RefusesPlanEditedSinceSpawn`: the batch's card edited after the recovery spawned → `ErrFingerprintMismatch`, the record not terminal.
- `verbs_test.go`: `TestValidateCmd_RefusesOverviewEditWithoutRestamp`: `00-overview.md`'s `## verify:` edited under a run → non-zero exit naming `rebaseline`, `PlanFileHashes` and `PlanFingerprint` unchanged in `state.json`, and the next `Run` refuses with `ErrFingerprintMismatch`.
  The existing restamp test still passes on an unedited plan.
- `template_test.go`: the plan-drift section names `record-batch` and `recover-batch` with `plan_drifted` and names `lyx webster restore-plan`.

**ImpactSummary:** validate, record-batch and recovery refuse a plan edited since the run recorded it or since the batch began, instead of adopting the edit into the plan hashes; fingerprint-mismatch way forwards also name restore-plan, and the Master stencil stops on the new plan_drifted refusals.
