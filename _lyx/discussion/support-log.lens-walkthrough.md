```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: I ran each decision on one concrete case and checked its end state against the decision's own purpose.

**Findings**

1. **BLOCKING — D4, yield mechanism.** No existing path turns "InnerRun's check ended nothing" into "the process exits with the run left `running` and no cap spent".
   - Case: a loom child is running at Plan-Write with nothing changed, and dobby's probe tick runs a pass. The pass's InnerRun Call must return exactly Done, Stuck or Awaiting (shedengine doc, "Call must return exactly Done, Stuck or Awaiting and nothing else").
   - Stuck writes a history entry and Awaiting halts the run.
   - Ending through a cancelled Call is the only other route, and the Go driver reads that as an interrupted step. Under the re-invoke policy it re-steps in-process, so the pass loops instead of exiting. Over the repair cap it goes to the tackler (sheddriver doc lines 38–40: "an interrupted step under the re-invoke policy, below the repair cap: re-step", "an interrupted step over the repair cap ... the tackler decides").
   - So D4's own bound, "counts against no repair cap", cannot be reached from what the existing steps produce. The rejected alternative ("a yield needs only a process exit ... which a batten end already does") rests on a false premise.
   - The record has to name the mechanism: a shed or driver yield outcome, or something equivalent, with its bound.

2. **BLOCKING — D4 with D9, dead driver of a running child.** The blanket rule "a rule stated per batten process becomes per episode" combines with D9's driver-dead gate to leave a dead driver silent.
   - Case: a running child's driver dies at minute 10 and its status file does not change, so the episode is the same.
   - Today a running child whose spawn this batten process has not confirmed is respawned. The confirmation marker is keyed by pid, so a new process reads it as unconfirmed.
   - Under D4's per-episode rule the confirmation persists across passes, so no pass respawns the child.
   - D9 sends driver-dead "only after the pass's respawn or revive of the driver failed". No respawn is attempted, so no notice is sent and the run sits dead.
   - That misses the entry's "the same pass finds dead drivers".
   - The record has to say that confirmation stays per pass (each pass re-checks strand and locks and respawns a driverless child), with a bound on repeated respawns. The driver-dead notice should follow a failed respawn.

3. **MEDIUM — D3/D4, enumerating runs.** The record never says where a dobby gets the set of in-flight runs it drives.
   - Case: a pass runs Land, which prunes the task from the board. The machine restarts before Worktree-Teardown runs.
   - The restarted dobby runs a pass "for every run in flight" (D4). If that set comes from board run statuses, the pruned task is not in it, so the pair is never torn down.
   - The same gap leaves open how a live dobby learns of a run that `lyx beam land` or a hand-run `lyx batten run` started.
   - The source has to be stated, for example the local batten status files of runs whose parent is this worktree, rather than the board.

4. **MEDIUM — D4 trigger list with D13, self-triggered loop.** Dobby runs a pass whenever batten's own status file is written, and a pass resuming at Run-Shed may itself write that file.
   - D4 promises only "no history entry and no stop report", not "no status write".
   - Case: pass P1 yields, its write fires fswatch, and P2 runs one second later (D13's coalescing). That repeats forever, one pass per second per idle run.
   - Either the yield must write nothing to batten's status file, or the trigger must ignore the pass's own writes.

5. **MEDIUM — D4 hand-run with D5 and the new PATTERN entry.** A hand-run `lyx batten run <slug>` on a fresh task is a start.
   - Case: the operator runs `lyx batten run T` on a task whose status is `draft`. The first pass claims it, overwriting `draft` with a run status.
   - That contradicts D5's bound ("a task is started only by its parent's dobby, only from an empty status") and `PATTERN-dobby-starts-runs`.
   - Either batten's start guard refuses a draft, or a fresh start not made by dobby, naming the way forward, or the bound and the PATTERN text must allow the hand start.

6. **MEDIUM — D5, refusal retry key.** "Not tried again until its board entry's `revised` time changes" can block a task for good.
   - The board doc says "a status change revises nothing", so the operator cannot nudge the task with `set-status`.
   - Case: the first pass is refused because origin is unreachable. That is a transient refusal, but the task is never retried until someone edits its scope.
   - Transient refusals need a timed retry, or a key other than `revised`.

7. **MEDIUM — D9, api-error revive with no cap.**
   - Case: a provider outage. The pass sends a revive line, the session becomes active, and the episode ends, because episodes are keyed by the newest agent activity.
   - The next API error is a new episode, which gets another revive and no notice.
   - During a long outage the orch is never told, and revive lines repeat every few minutes.
   - The notice should go after N consecutive revived episodes, or on a backoff.

8. **MEDIUM — D1/D2, adopt.** `lyx adopt <beam>` moves a beam's ownership, but nothing starts the new owner's dobby or stops the old machine's.
   - Case: beam B is adopted from machine A to machine C. Runs under B that C now owns have no running dobby until someone runs `lyx beam start` on C, which breaks "a run's parent always has a running dobby".
   - A's dobby stays alive, driving nothing.
   - Adopt should end with `lyx beam start`, and a dobby whose beam is owned elsewhere should exit.

9. **LOW — D5, `max_runs` counting.** The record does not say whether halted or awaiting runs count as in flight. If they do, three runs waiting on the orch stop every new start; this should be stated.

10. **LOW — D12, pause during Land.** With dobby still running probe passes under a pause, a run whose child finishes enters Land under the pause and halts blocked, needing `lyx batten goto --to Land`.
    - The pass dobby runs when the pause lifts does not bring that run back, so D12's "shows whether the run came back" does not hold for it.
    - Dobby could hold off passes for a done child while a pause covers it.

11. **LOW — D10, tidy across a re-exec.** "One tidy agent at a time" needs the in-flight tidy recorded in dobby's state record, so a re-exec or restart does not spawn a second one.

**Per decision (case used: purpose met or not)**

- D1 (beam B lands; teardown's `runproc.End(B)` ends B's dobby): met; adopt is not covered (finding 8).
- D2 (`lyx orch start` after a restart starts every dobby): met for owned beams; not met after an adopt (finding 8).
- D3 (a task under grouping project G resolves to the prime's dobby): met; how in-flight runs are enumerated is not stated (finding 3).
- D4 (idle child, probe pass): not met (findings 1, 2, 4).
- D5 (an empty-status task with dependencies done and one run in flight is started): met; the hand start and retry issues are findings 5, 6 and 9.
- D6 (the Land notice names no ready task): met, though it overlaps D9's dropping of `landed`.
- D7 (dobby's start refusal goes to the nearest orch): met.
- D8 (a batten Land failure the tackler escalates goes to the arbiter inside the pass; a loom override of 0 turns the arbiter off): met.
- D9 (API outage; dead driver): not met (findings 2 and 7).
- D10 (fifth landing triggers a tidy and the report path is posted): met, apart from finding 11.
- D11 (the tip sets one key the parent lacks; the notice names base, tip and parent values): met.
- D12 (lifted hub pause): met, except a Land halted under the pause (finding 10).
- D13 (a released draft wakes dobby through an fswatch event on `board.json`): met.

**Tool calls**

- Grepped the sheddriver doc for interrupted-step handling and the shedrun and loomcli docs for the watched marker: my context lacked how the driver treats an interrupted step and who reads the marker.
- Grepped for the watched marker's consumers and batten's verify retry: I needed to confirm that `loomcli` reads the marker's pid as a liveness check on each start.
