```yaml
lens: code-correctness
outcome: reported
```

Task: the code-correctness lens, running one concrete case through each changed function, branch and error path.

**Per-function list**

- `ParseReport` / `validateReportFields` (`report.go`):
  - `NEEDS_PARENT` with `need: "  "` is refused, since it is trimmed to empty.
  - `OK` with a `need` is refused.
  - `NEEDS_PARENT` with an empty `head_sha` hits the head_sha error first.
  - An old report with no `need` key reads unchanged.
  - Correct.
- `distill` (`digest.go`): `NEEDS_PARENT` gives `Waiting` with `Need`, `FAILED` gives `Stuck`, `OK` gives `Done`. The `default` arm also maps an unknown status to `Stuck`, but `ParseReport` rejects those first. Correct.
- `waitsOnParent` / `recordWaiting` / `resumeStep` / `waitingNeed` / `waitingRefusal` (`waiting.go`):
  - A nil `bs`, a terminal `bs` and a non-waiting `bs` all give false.
  - `recordWaiting` archives the report, so the later `begin-batch` existing-report guard (`beginbatch.go:~540`) finds none.
  - `Digest` is set and `Terminal` stays false, with `Recoveries`, `StartSHA` and the cursor untouched.
  - A nil `Digest` gives an empty need and nothing panics.
  - Correct.
- `RecordBatch` waiting guard (`recordbatch.go:305`): a waiting fork or recovery batch refuses, naming `begin-batch NN` or `recover-batch NN` respectively. It sits before the `Kind != "fork"` check, so the recovery case gets the right verb. Correct.
- `RecordBatch` `NEEDS_PARENT` branch (`recordbatch.go:~523`):
  - It runs after the head resolve, the mid-merge check, the head reconcile, and the policy-warning recording.
  - It skips the OK-only card-verify rerun.
  - It returns before `postBatchChecks`.
  - The caller saves state.
  - A correctness finding raised earlier, before the report is read, still fails the batch. That is plausible and not a defect.
  - Correct.
- `BeginBatch` re-begin of a waiting fork batch (`beginbatch.go:~620`): the new record replaces `bs` and carries over the start SHA, recoveries and transcripts, with no waiting status carried. The report is already archived, so the existing-report refusal is not hit. Correct.
- `recoverSpawn` count (`recoverbatch.go:382`): a waiting `prior` spawns without incrementing `recoveries`, and the fresh record has an empty `Status`, so `waitsOnParent` is false afterwards. Correct.
- `RecoverSpawnOrAttach` (`recoverbatch.go:487`, `496`):
  - A waiting recovery record is not attached to.
  - At `Recoveries == 2` the exhaustion check is skipped and the spawn is uncounted.
  - A waiting fork-kind record is also treated as waiting. That is harmless.
  - Correct.
- `PersistRecoveryTerminal` waiting branch (`recoverbatch.go:627`):
  - It returns before `checkRecoveredSuspects`, the pass, and the `Terminal` / `CurrentBatch` clearing, so the batch stays in flight.
  - `recordWaiting` archives the report.
  - The strand's `StrandGUID` stays on the record, and the next spawn path stops a recorded strand.
  - `awaitTerminal` still runs `reconcileReportHead` on a waiting digest first, so a head-reconcile error is returned before the record becomes waiting. That is consistent with the done and stuck paths. Correct.
- `webstercli` recover-batch switch (`recoverbatch.go:358`):
  - Waiting exits 0 and done exits 0.
  - Stuck and dead set `recovery_session` from `lastRecoverySession`.
  - A nil record or no sessions gives an empty string, so the message reads "session )". That is cosmetic.
  - On a waiting envelope `recovery_retry` is false, because `RecoveryRetry` requires a terminal dead record. The envelope also carries `status: waiting` and `need`, which Master's stencil reads first, so this is correct but potentially confusing.
- `RenderProgress` (`render.go:540`): `waitsOnParent(bs)` is called before the `ok` check, but it handles nil. A waiting batch gets a line with its need and resume verb. Correct.
- `deadEndBatchError` (`runlevel.go`):
  - A terminal dead batch with `recoveries 0` gives retry true and no refusal.
  - A terminal dead batch with `recoveries 2` and no amendment is refused by the first case.
  - A terminal failed batch with `recoveries < 2` is not refused.
  - A nil `bs` and a done batch are skipped.
  - The check is placed after the pending-audit check and before Master is spawned.
  - Correct for the stated cases. The mismatch below is F1.
- `ErrBatchDeadEnd` and the adapter (`webster.go:79`): the mapping to `Stuck` is correct.
- Shed `BudgetExempt` (`run.go:~157`): `def.OnStuck == ""` makes a blocking Stuck exempt. `appendOrFold` folds identical repeats, and a goto or reroute between them still separates the entries. I could not confirm that exempt entries never reach a budget comparison; I did not open `appendOrFold` or the budget code. Plausibly correct.

**Findings**

- F1, LOW, design. `internal/websterengine/runlevel.go:~1180` (`deadEndBatchError`, second switch case): a dead recovery at the recovery cap with an unrendered amendment passes the first case but is refused by the second.
  - `RecoveryRetry` returns false whenever `recoveries >= maxRecoveries`, whatever the amendment.
  - `RecoverSpawnOrAttach` (`recoverbatch.go:496`) allows a spawn at the cap when `rendersNewAmendment`.
  - So `Run` refuses a batch that a `recover-batch` call could still recover, which conflicts with the decision's "exhausted … no unrendered amendment" bound.
  - Fix: in the dead case, skip the refusal when `rendersNewAmendment(bs)`.
- F2, LOW, design. `internal/websterengine/runlevel.go:~1172` (`deadEndBatchError`): `RecoveryRetry` returns false on a failed git read (logged at Warn), so a transient git error at `Run` entry is read as a dead end. The refusal text then claims "its recovery earns no retry" and blocks the run on a transient fault. The git-error return value is discarded (`retry, _`). I think the exposure is narrow.

**Tool calls (4 of 6)**
1. `beginbatch.go:435-640`: to confirm a re-begin of a waiting fork batch replaces the record and does not carry the waiting status.
2. `recordbatch.go:440-520` and `recoverbatch.go:560-660`: the checks before the new `NEEDS_PARENT` branch and the `PersistRecoveryTerminal` flow.
3. `grep` of `classify.go`, `recoverbatch.go` and `webstercli/recoverbatch.go` for status and terminal uses, plus `recoverbatch.go:405-440`: `RecoveryRetry` semantics.
4. `recoverbatch.go:700-760` and `840-870`: terminal persistence and the `awaitTerminal` tail.
5. `grep` of `webstercli/recordbatch.go` and `recoverbatch.go` for digest status and commit points (the call was a single `Bash`).
