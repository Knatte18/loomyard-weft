```yaml
lens: code-affected-set
outcome: reported
```

Task: lens code-affected-set. For each change, I list what else reads or asserts it, and whether the diff covers it.

**Per-change list**

1. **Report status `NEEDS_PARENT` and `need`.**
   - Set found: `ParseReport`, `distill`, `WriteReport`, `Report`, the implementer body, the master template, the websterengine `doc.go`, `report.go` and `digest.go`. `webster-prefix-recovery.md` and `webster-prefix-fork.md` also mention `head_sha`, and I did not read them.
   - Covered: the implementer body appears to reach recovery strands through the prefixes, and the diff updates all of the rest.
   - Not covered: finding F1 below.
2. **Waiting batch: non-terminal, `Status` "waiting".**
   - Set found: every reader of `bs.Status`/`Terminal`:
     - `BeginBatch` (`beginbatch.go:662`) builds a fresh `BatchState`, so `Status` resets on re-begin. It keeps `Recoveries`, `StartSHA` and `RecoverySessions`.
     - `existingReportRemedy`: unreachable, because the report is archived.
     - `reset.go:341` and `rebaseline.go:332`: both gate on `Terminal`.
     - `runlevel.go:329` is the verify-fixer path.
     - `deadEndBatchError` skips non-terminal batches.
     - `RenderProgress` handled in the diff. `recoverSpawn`, `RecoverSpawnOrAttach` and `PersistRecoveryTerminal` handled in the diff.
   - Covered, except F2.
3. **`recover-batch` exit 0 on waiting; `recovery_session` in the envelope.**
   - Set found: `recoverBatchCmd`, `recoveryNotDoneMessage` and its callers, the `Long` help, the master template's exit-code sentence. All updated.
   - Covered.
4. **`ErrBatchDeadEnd`.**
   - Set found: `Run`, the shed adapter `webster.go` plus its `doc.go`, `webstercli/run.go` help, the websterengine `doc.go`. All updated.
   - Covered.
5. **`BudgetExempt` for a Stuck at a row with no `OnStuck`.**
   - Set found: `run.go`, the `HistoryEntry` comment in `status.go`, the shedengine `doc.go`. All updated.
   - I did not check the `appendOrFold` consumers; that is outside my 6 calls.
6. **Stencil wording.**
   - The implementer body remaps the refused-verb sentence and the "reports status: FAILED and the orch runs it" clause. The master template adds a start-of-run `waiting` rung and a ladder rung, and does not drop or weaken other rules.
   - Covered, except F1.

**Findings**

- **F1, LOW, consistency.** `internal/fabriccli/roleguard.go:59` tells a webster implementer "stop and report status: FAILED so the run ends stuck and the orch runs it". The sentence is webster-specific and is not the quoted clause the body remaps (`webster-body-implementer.md:125`), so an implementer meets a refusal that contradicts the body's `NEEDS_PARENT` instruction. The same goes for `internal/websterengine/gitwrap.go:47` and `internal/webstercli/beginbatch.go:28`, which hold the shared clause and are mapped by the body. Only the roleguard wording is not.
  - The decision record puts the shared clause out of scope.
  - roleguard is the direct fabric refusal a webster strand hits, and it is also covered by `PATTERN-refusal-way-forward`'s "no refusal names a step agents' settings deny".
  - Fix: reword the roleguard message to "report status: NEEDS_PARENT naming the verb" for webster strands, or leave it and record it as a known gap.
- **F2, NIT, consistency.** `internal/websterengine/runlevel.go:1019`: the `outcome: done` check reports a waiting batch as "(begun, not recorded terminal)". That hides the cause. Fix: add `case waitsOnParent(bs):` before it, naming the need and `resumeStep`.

**Tool calls**
1. `sed`/`grep` over `beginbatch.go` and the `FAILED` search: no manifest content on re-begin or on other `FAILED` sites.
2. Grep of `.Status` readers: to find the `Status` readers outside the diff.
3. `beginbatch.go` 598–680: to see the re-begin `BatchState` construction.
4. Grep of the stencils, `runlevel.go` 1010–1026 and 320–336, and docs: to check the stencil, run-exit and verify-fixer readers.
5. `ls` of the stencils directory and grep of the recovery prefix: to check how the recovery strand gets the report contract.
