```yaml
lens: guard-reach
outcome: reported
```

Guard-reach lens: whether each guard, refusal, branch or test row the cards add can be reached from a real entry point.

**Findings**

1. **BLOCKING** (Card 13, D12 delivery on attach): the delivery that card 13 prescribes "before it waits" can never run before the wait.
   - `internal/shuttleengine#Runner.AttachGated` attaches and waits in one call (`attachAndWait`, internal/shuttleengine/attach.go:87-94).
   - No point exists between the attach and the wait. Without a recorded guid, the strand's guid is only known from `Result.StrandGUID` once the strand has ended.
   - So the attached live strand never gets the undelivered decisions, and `TestRun_Resume`'s "the attached strand gets the undelivered decisions once" row cannot pass as written.
   - The ordering D12 requires needs `ProbeGated`: take the handle, deliver through `Run.StrandGUID()`, then call `Run.Wait()`.
   - The same applies to a fixer that `gatefixer.SpawnOrAttach` attaches: it returns the record, so it can deliver before `Await`. That one is fine, but card 13 does not say so.

2. **BLOCKING** (Card 11, 18 and 21, `RunDeps.Deliver`): nothing fills `Deliver` in production, so every D12 send on attach is unreachable.
   - Card 11 says "Deliver … (card 21 fills it)".
   - Card 21 edits only `decisionNoticeInline`, `heldBackReason` and `notifyLiveRuns`. It does not list `internal/loomcli#loomCLI.wire`.
   - Card 18 wires `WeaverDeps` without `Deliver` and adds `Deliver` to `nilLegal` as optional.
   - With a nil `Deliver`, card 13's send branch never runs.

3. **BLOCKING** (Card 21, `RecordDelivery`): delivery recording is keyed by strand guid, but the notice loop only has strand names.
   - `notifyLiveRuns` gets strand names from `liveRunStrands` (internal/loomcli/decision.go:1724-1745, `reading.StrandName`). It never has a guid.
   - `RecordDelivery(anchorRoot, strandGUID, headings)` matches attempts and fixers by guid.
   - The matching branch is unreachable from `notifyLiveRuns`, so no delivery is ever recorded.
   - A resumed attach would then re-send entries the live notice already delivered, breaking the "not yet delivered" criterion.
   - Fix: key it by strand name, or resolve the name to a guid through reed or shuttle state.

4. **BLOCKING** (Card 22, rework and plan-write archive): Weaver's archived run directory leaves unstaged deletions in the weft.
   - The archive seam now also moves `_lym/weaver/` into the `weaver` slot.
   - The plan commit's pathspec, `internal/loomcli#planCommitPathspecWith` (wiring.go:149-157), stages the moved webster deletions only.
   - No card edits it, so after a rotation over a Weaver run the tracked `_lym/weaver` deletions stay uncommitted.
   - That untracked dirt is the condition `planCommitPathspecWith` exists to prevent, and here it recurs unguarded for Weaver's directory.

5. **MEDIUM** (Card 14, `Reset` after a failed attempt): the ordinary state Reset meets makes it refuse, and no card names or tests that refusal.
   - An attempt failed on an unclean tree is the D6 case, and it leaves uncommitted tracked edits.
   - `ResetCode` is fabric's `ResetPairCode`, which refuses dirty tracked paths outside `ownPaths` (`dirtyTrackedExcept(ownPaths)`, internal/fabricengine/destroy.go:2325-2333).
   - Card 14's `ResetCode func(sha string) error` passes no own paths.
   - Card 14 names neither that refusal nor its way forward, and no `TestReset` row reaches it.
   - Webster's `PlanReset` handles the same state itself (reset.go:238-251).

6. **MEDIUM** (Cards 12 and 13, a plan-conflict stop then re-entry): what a plan-conflict stop leaves behind for re-entry is unspecified.
   - Card 12 records the partition (`RecordBeginPartition`) before `ResolveAtDispatch`, and records the batch's card set and start commit only after it passes.
   - Card 12 does not say whether a conflict stop sets `StopRecord`.
   - If it does, card 13's `reenter` runs the post-batch checks on a batch with no start commit recorded.
   - Its partition check also refuses the follow-up card Plan-Review inserts after the last begun batch: the recorded partition holds the fresh pending batches, and the new card is in none of them.
   - The reroute D13 exists for would then halt at re-entry.
   - If it does not, card 13 needs to say a reroute re-entry takes the normal begin path.
   - No `TestRun_Reentry` row covers re-entry after a reroute.

7. **LOW** (Card 17, D1 guard): the guarded site recurs unguarded on Webster's CLI path.
   - The guard sits only in `websterProducer.Call` and `weaverProducer.Call`.
   - `lym webster run` and `begin-batch`, run by hand in a hub worktree with Weaver begun, reach `websterengine.Run` and `RecordPartition` without it (`internal/webstercli#websterCLI.beginBatchCmd`).
   - The both-begun state card 19 and card 20 refuse on is reachable only that way. Through the rows, the guard keeps both from beginning, so their both-begun rows test a planted state only.

8. **LOW** (Card 9, `classifyAttempt`): one row tests a state shuttle cannot produce.
   - The row is "a run shuttle reports finished with no report".
   - Shuttle's `OutcomeDone` is decided by the output files existing (internal/shuttleengine/attach.go:518-536, spec.go:26-31). For a run whose only output file is the report, done with no report does not occur.
   - The row exercises only a fake.

9. **LOW** (Card 10, head check): `recordBatch`'s strict check fails valid attempts.
   - The check requires the report's `head_sha` to equal HEAD.
   - Webster resolves an abbreviated head and tolerates a clean parent merge (`resolveReportHead` and `reconcileReportHead`, recordbatch.go:486-502).
   - Under Weaver, an attempt reporting a short SHA fails and climbs the ladder, as does one followed by a clean parent merge-in, which `lym fabric merge-in` invites with only a warning (card 23).

**Tool calls**
- Read `internal/loomcli/decision.go` lines 1720-1762 (`liveRunStrands`) and `internal/fabricengine/destroy.go` lines 2290-2370 (`ResetPairCode`), one Bash call. My context lacked whether live strands come back as names or guids, and whether `ResetPairCode` refuses dirty paths.
