# Card 47 — fabric-reference-correctness

**Edit:**
- `internal/websterengine/audit.go`
- `internal/websterengine/doc.go`
- `internal/websterengine/auditseverity_test.go`
- `internal/websterengine/recordbatch_test.go`
- `internal/websterengine/runlevel_test.go`

**Uses:**
- `internal/websterengine/recordbatch.go`
- `internal/websterengine/runlevel.go`

**Intent:**
Fix PR #321 round-3 finding C: the correctness/policy line for a fabric reference is drawn by a shell parser, and its read-only allowlist is a vocabulary an agent can step around.
The invariant already says agents never touch the fabric repo directly, so every fabric reference is correctness, whatever its command.

In `audit.go`:

- `ClassifyViolation` classes every `ClassFabricReference` finding as `AuditSeverityCorrectness`.
  Its doc comment drops the allowlist sentence and states the rule: an agent never touches the fabric repo, and a command that reaches it can rewrite run state that a re-run of the cards' verify commands cannot detect.
- `fabricReferenceDetail` loses its read-only branch: every fabric-reference Detail says the command can rewrite run state.
- Delete the shell parser and every table only it reads: `mutatingCommand`, `shellSegment`, `splitShell`, `readOnlySegment`, `steersPrograms`, `skipOptions`, `clusterEndsInValue`, `grepOpensPager`, `programName`, `stringSet` and the read-only and option sets declared beside them.
  Drop the `path` and `slices` imports if nothing else in the file uses them.
- `AuditViolation.Command` stays: the failed batch's digest and the run-exit stuck reason still quote it through Detail.

`doc.go`'s audit-findings paragraph says every fabric reference is correctness and removes the read-only allowlist rule.

Tests:

- `auditseverity_test.go`: delete `TestClassifyViolation_FabricReferenceByCommand`.
  In `TestClassifyViolation`, the `fabric-reference` row keeps its correctness verdict, and a second row with `Command: "cat <dir>/webster/state.json"` is correctness too.
- `recordbatch_test.go`: every test that used a fabric reference to reach the policy path switches to a policy finding that still exists.
  `forkFabricRefAudit` becomes a fork transcript with `AgentCalls: 1` (`nested-agent`), and `TestRecordBatch_RetryNeverDuplicatesWarning` and `TestRecordBatch_RetryFailingVerifyFailsNamingEarlierWarning` replace their `ParentBashCommands` with `NamedSpawns: 1` (`named-spawn`), asserting `audit warning (named-spawn)`.
  Rename each test whose name says fabric reference to name the finding it now uses.
  `TestRecordBatch_Regression20260930_ForkAuditFalsePositive` keeps the incident's fabric reference (`cat FABRICREF/webster/state.json`) and pins what keeps it from wedging now: the first call fails the batch with its report archived and names `lyx webster recover-batch 01`, a second `RecordBatch` call returns no audit refusal, and `RecoverSpawnOrAttach` spawns.
  `TestRecordBatch_UnlistedWriterFabricReferenceFailsBatch` folds into `TestRecordBatch_MutatingFabricReferenceFailsBatch`, which becomes a table over a read-only command (`cat`), a mutating one (`git ... checkout`) and an unlisted writer (`python3 -c ...`), each failing the batch with the command in the digest.
- `runlevel_test.go`: `TestRun_DoneWithFabricReferenceInIntegrationForkWarns` becomes `TestRun_DoneWithNestedAgentInIntegrationForkWarns`, with an integration fork transcript carrying `AgentCalls: 1` and the same run-level warning assertions.
  Add `TestRun_FabricReferenceInIntegrationForkIsStuck`: the integration fork ran `cat FABRICREF/webster/state.json` → the run ends `stuck`, the stuck reason quotes the command, and `state.json` carries one pending finding with no path.

**ImpactSummary:** Every fabric-reference finding becomes correctness, failing its batch or holding run exit, and the shell parser behind the old read-only split is deleted with its tests.
