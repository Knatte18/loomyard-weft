---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/websterengine/runlevel.go:326
    summary: run-entry reclaim stops a non-terminal cold fixer but leaves its record non-terminal, so the next fix-verify attaches to the stopped strand and the run's one cold attempt is spent as dead
    fix: In reclaimEntryTimeStrands, after StopStrand succeeds for a record whose Status is empty, set rec.Status = DigestStatusDead and rec.DeadReason = DeadReasonDied, so FixVerifySpawnOrAttach, which attaches only to a last record with an empty Status, spawns a fresh cold fixer on the next gate failure. Make sure the state phase saves st after the reclaim (check how Run persists st after reclaimEntryTimeStrands; add a SaveState there if it does not already). Extend the existing run-entry reclaim test in runlevel_test.go with an assertion that the leftover fixer's record is terminal dead/died afterward and that a following FixVerifySpawnOrAttach spawns instead of attaching.
  - id: F2
    severity: MEDIUM
    class: consistency
    location: contracts/stencils/webster/webster-body-verify-fix.md:27
    summary: shared fixer body tells both fixers the run-exit audit flags a write under the plan directory or _lyx, but the audit only covers forks of Master's session, so a cold fixer's write there is unaudited
    fix: In webster-body-verify-fix.md reword the bullet "Never touch the plan directory ... they sit outside every code commit, and the run-exit audit flags a fixer that writes there" to a claim true for both fixers, for example "they sit outside every code commit, and a write there corrupts the run's own record". In internal/websterengine/doc.go, in "The cold verify-gate fixer" section, add one sentence stating that the run-exit audit reads the forks of Master's session only, so a cold fixer's own writes are checked only through the contract-file evidence loadRunWrites collects, as a recovery strand's are. Keep the fork-side sentences in "The verify-gate fixer fork" as they are.
  - id: F3
    severity: LOW
    class: consistency
    location: internal/websterengine/contractevidence.go:45
    summary: an uncleared contract file last written by a cold verify-gate fixer is reported as written by a recovery session, naming the wrong session kind to the operator
    fix: Make the writer label say what the Recoveries list now holds. Either rename writerRecovery's text to cover both ("a recovery or verify-fixer session") and update the doc comments on RunWrites.Recoveries, writerRecovery and contractFileStatus to match, or keep the label and split the fixer sessions into their own RunWrites field with its own writer constant. Update any test that pins the text "a recovery session" to the new text.
  - id: F4
    severity: LOW
    class: consistency
    location: internal/websterengine/template_test.go:902
    summary: new cold-fixer prompt row asserts the stencil's static prose "as a foreground Bash call", copied from the fork row at template_test.go:889
    fix: In the "cold fixer prompt names the gate report, the reply file and the contract files" row, change the contains entry "run `" + testFixGate + "` as a foreground Bash call" to testFixGate alone, which pins the filled fix_gate value without pinning the surrounding stencil wording.
---

### [MEDIUM] Run-entry reclaim leaves a stopped cold fixer non-terminal

**Location:** `internal/websterengine/runlevel.go:326`
**Issue:** `reclaimEntryTimeStrands` stops every `State.VerifyFixers` record whose `Status` is empty, then leaves the record as it was.
`FixVerifySpawnOrAttach` attaches to the last record while its `Status` is empty, so after a crash and re-entry the first cold-routed failure makes `fix-verify` attach to the strand the reclaim just stopped.
`FixVerifyAwait` then classifies it dead (`asking` or `died`) at once, and Merriam ends its turn with no fixer having worked.
The cold attempt is the escalation this change exists to add and an untuned hub gets exactly one, so a stale record spends the whole escalation.
The batch-recovery path has the same record shape, but its attempts are bounded by `maxRecoveries` with a fresh spawn after a dead record, so the cost there is lower.

### [MEDIUM] The shared body claims an audit the cold fixer is not under

**Location:** `contracts/stencils/webster/webster-body-verify-fix.md:27`
**Issue:** the body, now read by both fixers, says the run-exit audit flags a fixer that writes under the plan directory or `_lyx`.
`runExitAuditCrossCheck` runs `CheckFork` over `result.ForkAudit.Forks`, the forks of Master's session, and `CheckParent` over Master.
The cold fixer is a separate strand, so its own writes reach only `loadRunWrites`, which clears or flags Merriam's two contract files and nothing else.
A cold fixer that edits the plan directory or `_lyx` is therefore not caught, and the stencil tells it otherwise.
`doc.go`'s "The verify-gate fixer fork" section states the audit for the fork correctly, and the new cold section is silent on the gap, so the decision record's claim that the cold fixer passes through the same run-exit audit is not borne out for its non-commit writes.

### [LOW] Cold fixer writes are reported as a recovery session's

**Location:** `internal/websterengine/contractevidence.go:45`
**Issue:** `recordedSessions` lists the fixer sessions beside the recovery sessions, so `RunWrites.Recoveries` and `contractFileStatus` report a fixer's write to a contract file as `writerRecovery`, "a recovery session".
The operator reading the finding is sent looking for a recovery strand that never ran.

### [LOW] New prompt row asserts static stencil prose

**Location:** `internal/websterengine/template_test.go:902`
**Issue:** the new cold-fixer row asserts `"run `" + testFixGate + "` as a foreground Bash call"`, which pins the stencil's own wording around the filled marker.
The same assertion already sits in the fork row at `template_test.go:889`, so the new row adds a second assertion on static prose; the filled `fix_gate` value is the behavior.
