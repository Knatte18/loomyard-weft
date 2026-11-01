```yaml
lens: existing-paths
outcome: reported
```

Task: existing-paths lens. For each state the record introduces or changes, trace the code that already writes or reads it, and judge whether the decision survives.

**Paths traced**

- **D4 hold (`HoldCaller` slug entry)**
  - Survives:
    - dobby's `startReady` → `PauseCovers` (`starts.go:87`): released tasks wait.
    - dobby's owner-pause lift (`beamcli/dobbydeps.go:315`): it selects `OwnerCaller` only.
    - hibernate wake (`hibernateengine/wake.go:58`): it lifts only its own entry.
    - orch pause (`orchengine/pause.go`): it never lifts slug entries.
  - Breaks the declared wording: `lym batten cancel` → `liftSlugPauses` (`cancelcmd.go:454-486`) lifts every caller of the slug's scope.
- **D4 slots**: `slotsTaken`/`runsInFlight` (`starts.go:143-168`) count held listed runs. D8's bypass is needed, and the record states it. Survives.
- **D5/D8 removal through `lym batten cancel --remove`**
  - `refuseNothingLeft` (`cancel.go:554-598`) breaks it for a fix task that has no claim, record or pair.
  - `clearRunClaim` (`cancelcmd.go:489-503`) writes draft over a run status. Survives as D8 states.
- **D6 slug reuse**: `dobbyreads.go:288-294` → `doneSlugRefusal` (`arm.go:865`) refuses a re-filed slug whose run record reads done. Breaks.
- **D6/D8 "open" over a halted fix**: a halt is a run status such as `blocked · <row>` (`shedengine/status.go:18-23`, `runlock.go:23`), so the halted fix task reads open. Breaks the D6 loop.
- **D8 filing in one write**: `applyDraftRule` (`draft.go:37-60`, `doc.go:245-248`) turns a new task with an empty status into draft. Breaks.
- **D8 payload refusal**: `checkBeamPayload` is called from `UpsertTask`, `UpsertTasksBatch` and `MergeTasks` (`store.go:507,842,1021`). D8's mirror fits. Survives.
- **D8 run-agent guard**: only boardcli sets `StrandName` (`boardcli/cli.go:161`, `boardengine/config.go:56`), so an in-process write with no strand name is not judged. The claim holds.
- **D8 rename**: rename keeps every field under a new slug (`boardengine/doc.go` Rename section), but D8's record is keyed by slug. Breaks.
- **D8 `on_land` record and healing**: the old machine's dobby still ticks for runs owned elsewhere (`owned.go:17`). D8's healing is not gated on ownership. Breaks.
- **D9 stop**: `darnwrite.go:144-146` appends `darnStopWayForward` to every stop report. Breaks D6's "never `lym loom resume`".
- **D7 `halted`**: shed states are running, paused, done, blocked, failed and awaiting (`status.go:18-23`). None is "halted". Misread.
- **D7 `running_here`**: a cancel on the old machine removes the run's records and writes draft (cancel help, `clearRunClaim`). Misread.

**Findings**

1. **BLOCKING · design · D6 slug rule.** Every try re-files the same `adopt-fix-<subject>` slug.
   - Once try 1 has landed, its run status reads done. Dobby's task view then carries `doneSlugRefusal` ("already completed … `lym batten cleanup --apply` … or give the new task a different slug"). Dobby never starts try 2, so the loop dies at the second failed check.
   - Evidence: `internal/battencli/dobbyreads.go:288-294`, `internal/battencli/arm.go:865-869`, `internal/dobbyengine/starts.go:93-96`.
2. **BLOCKING · design · D6/D8 open front task.** A fix task halted on a machine failure keeps a run-form status such as `blocked · <row>`, so D8 reads it as open.
   - The operator re-runs `lym adopt <subject>`, as D6 and D9 direct. If the check is still red, D6 files nothing, and its notice says to re-run once the halted task lands. It never lands, and `lym loom resume` is ruled out.
   - Dobby starts no other task there, so the subject stays stuck.
   - D6 removes only a draft fix before filing, never a halted one.
   - Evidence: `internal/boardengine/runlock.go:21-24`, `internal/boardengine/doc.go:220-221`, `internal/shedengine/status.go:18-23`.
3. **BLOCKING · design · D8 removal step 1.** `lym batten cancel <slug> --remove` refuses with "nothing of <slug> is left to cancel" when no record, claim, pair or process remains.
   - That holds for a released fix task that never started, and for a draft fix left by a hand `cancel --remove`. That second command is the way forward D5 itself names.
   - D8 stops the removal at a failed step. D6's removal of a draft fix before filing then never completes, and the record says nothing of whether the filing goes ahead. If it does, it meets the same slug.
   - Evidence: `internal/battencli/cancel.go:554-598`, `internal/battencli/cancel_integration_test.go:139`.
4. **BLOCKING · design · D8 healing and `on_land` across machines.** D8 has the dobby record any open front entry it sees with no local record, and run its action once the entry is done or gone. Nothing gates this on ownership.
   - The old machine's dobby of the same worktree still ticks; it pauses runs owned elsewhere and keeps going (`owned.go:17-57`).
   - It therefore records the new machine's fix task and runs `lym adopt check <subject>` on the old machine when the fix lands. A red check there files a fix task into a subject this machine no longer owns.
   - Evidence: `internal/dobbyengine/owned.go:14-17`, decision record D8 bullet 5.
5. **MEDIUM · design · D8 filing in one write.** "One `boardengine` write of a released task" is a false premise.
   - The write critical section gives every new task with an empty status `draft`. Release takes a second write that sets the status to null.
   - Between the two writes the front task reads closed, and D8's healing only sees open entries. The record names neither the second write nor an exemption from the draft rule.
   - Evidence: `internal/boardengine/draft.go:37-60`, `internal/boardengine/doc.go:245-248`.
6. **MEDIUM · design · D9/D6 way forward.** `darnwrite.go` appends `darnStopWayForward` ("record the answer with `lym loom decision add`, then run `lym loom resume`") to every darn stop's reason.
   - So the machine-failure halt names `lym loom resume`, against D6's "never `lym loom resume`" and D7's "naming the way forward its stop report gives".
   - D9 does not change the appended text for this stop.
   - Evidence: `internal/loomshed/darnwrite.go:30,138-146`.
7. **MEDIUM · design · D7 `running_here`.** A run cancelled on the old machine reads `running_here`.
   - Cancel removes the run's records and writes draft, so after the reconcile the entry is present and not done. The run is also not in the moved runs read after the reconcile, so it is not held.
   - It falls through to "any other run this machine owns", which reports a cancelled run as running.
   - Evidence: `internal/battencli/cancelcmd.go:52-56` (help) and `489-503`, `internal/adoptcli/adopt.go:186,216`.
8. **LOW · design · D8 rename.** `lym board rename` is not refused on a released front task, and it carries `front` and `on_land` to the new slug.
   - D8's record, keyed by slug, then reads the old slug gone and runs `adopt-check`.
   - The renamed entry is healed as a new front task and runs its action again when it lands. D8's bound only covers an entry removed by hand.
   - Evidence: `internal/boardengine/doc.go`, Rename section ("the entry keeps … every other field").
9. **LOW · consistency · D4 against declared `PATTERN-adopt-hold`.** `lym batten cancel` lifts every live entry of the slug's own scope, including a `HoldCaller` entry.
   - The declared text says only the subject's green check, or `lym tender resume`, lifts a hold. D4 does not name the cancel path.
   - Evidence: `internal/battencli/cancelcmd.go:320,454-486`.
10. **LOW · consistency · D7 `halted`.** No batten run record reads "halted"; the shed states are running, paused, done, blocked, failed and awaiting.
    - D7 does not name which record states map to `halted`.
    - Evidence: `internal/shedengine/status.go:18-23`.

**Tool calls**

1. `liftSlugPauses` and the list of pause-record writers: the manifest had cancel's `LiftPauses` seam but not its body or the other writers.
2. Caller selection in the hibernate, orch, cleanup and dobby-owner lifters: to check whether any of them lifts `HoldCaller`.
3. `orchengine/pause.go` clearing and `slugPauseRefs`: to confirm the orch clears no slug entry and that cancel lifts every caller.
4. Hibernate's `Lift` wiring and wake caller matching: to confirm the wake lifts only its own entry.
