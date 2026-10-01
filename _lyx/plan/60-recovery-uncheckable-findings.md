# Card 60 — recovery-uncheckable-findings

**Edit:**
- `internal/websterengine/state.go`
- `internal/websterengine/suspect.go`
- `internal/websterengine/batchfail.go`
- `internal/websterengine/recordbatch.go`
- `internal/websterengine/recoverbatch.go`
- `internal/websterengine/runlevel.go`
- `internal/websterengine/doc.go`
- `internal/webstercli/recoverbatch.go`
- `internal/websterengine/recordbatch_test.go`
- `internal/websterengine/recoverbatch_test.go`
- `internal/websterengine/runlevel_test.go`
- `internal/webstercli/verbs_test.go`
- `internal/websterengine/template_test.go`

**Prosa:**
- `contracts/stencils/webster/webster-template-master.md`

**Uses:**
- `internal/websterengine/audit.go`
- `internal/websterengine/auditledger.go`

**Intent:**
Fix PR #321 round-4 finding B.
`checkRecoveredSuspects` skips a finding with no path (a fabric reference) and a path it cannot check (the scratch pause flag, a write into a sibling worktree), and `failFromFindings` never records a pathless finding at all.
Master's own failure ladder runs `recover-batch`, so such a batch goes done with no evidence, although `accept-audit` refuses the same findings.
A finding the recovery check cannot verify now keeps the batch failed, and its way forward is `lyx webster run --fresh`, as for `accept-audit`.

- `state.go`: `BatchState` gains `Uncheckable []string` (`json:"uncheckable,omitempty"`), one entry per correctness finding the recovery check cannot verify: the path, or `<class>: <detail>` for a finding with no path.
- `suspect.go`: `uncheckableSuspects(geom Geometry, st *State, paths []string) ([]string, error)` returns the paths `checkRecoveredSuspects` cannot check: neither under `geom.PlanDir` with `st.PlanFileHashes` recorded, nor accepted by `trackedRel`.
- `batchfail.go`: `failBatchInput` gains `Uncheckable`, which `failBatch` records on the batch.
- `recordbatch.go`: before calling `failFromFindings` for correctness findings, `RecordBatch` builds the list from `uncheckableSuspects` over the suspect paths plus one entry per pathless correctness finding.
  `failFromFindings` gains an `uncheckable []string` parameter passed through to `failBatch`;
  its other callers pass nil.
- `recoverbatch.go`:
  - A new exported sentinel `ErrRecoveryNeedsFresh`.
  - `RecoverSpawnOrAttach` refuses before `recoverSpawn` when the prior record is terminal `failed` with a non-empty `Uncheckable`, mutating nothing and spawning nothing:
    `webster: batch NN failed on findings recovery cannot check: <entries>; way forward: reset the branch to the run's start commit <sha> with git and run "lyx webster run --fresh"`, `<sha>` from `runEvidenceBases` and omitted when unknown.
  - The `PersistRecoveryTerminal` doc comment drops "A path the check cannot verify is not checked" and points at the entry refusal.
- `runlevel.go`: `freshPendingDrop` treats a batch record with a non-empty `Uncheckable` like a pending finding.
  It is something to drop, so `--fresh` re-initialises on an unchanged plan;
  card 58's HEAD rule applies, the record's `SuspectPaths` join the path check, and each such batch adds the drop warning `--fresh dropped batch NN's uncheckable findings: <entries>`.
- `internal/webstercli/recoverbatch.go` emits `needs_fresh: true` with the error when it wraps `websterengine.ErrRecoveryNeedsFresh`.
- The Master stencil's failure ladder gains a rung after the `batch_failed` one: `recover-batch <NN>` refuses with `{"needs_fresh": true}` → write `outcome: stuck` with a `stuck_reason` quoting the refusal, and stop;
  do not call `recover-batch` for that batch again and do not begin the next batch.
  Its "Audit findings" section says a correctness finding recovery cannot check comes back from `recover-batch` this way.
  Keep semantic line breaks and change nothing else.
- `doc.go`'s recover-batch paragraph states the rule.

Tests:

- `recordbatch_test.go`:
  - `TestRecordBatch_FabricReferenceRecordsUncheckable`: a fork Bash command referencing the fabric → the batch fails with `Uncheckable` naming the `fabric-reference` finding.
  - `TestRecordBatch_ScratchParentWriteRecordsUncheckable`: a Master Edit to the scratch pause flag → `Uncheckable` names the path.
  - A tracked-file parent write leaves `Uncheckable` empty.
- `recoverbatch_test.go`: `TestRecoverSpawnOrAttach_RefusesUncheckableFindings`: both records above → `ErrRecoveryNeedsFresh` naming `lyx webster run --fresh`, the starter never called and the record unchanged.
- `runlevel_test.go`:
  - `TestRun_FreshDropsUncheckableBatch`: an unchanged plan, HEAD at the start commit and a batch failed with `Uncheckable` → the run re-initialises with the drop warning.
  - With HEAD one commit past the start → `ErrPendingAuditFindings` naming the start commit.
- `verbs_test.go`: `recover-batch` over such a record exits non-zero with `needs_fresh: true`.
- `template_test.go`: the failure ladder carries the `needs_fresh` rung.

**ImpactSummary:** A batch failed on a correctness finding the recovery check cannot verify records it, recover-batch refuses such a batch toward run --fresh instead of letting a recovery clear it, and --fresh drops such a batch under the same HEAD and path rules as pending findings.
