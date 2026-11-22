```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: I ran each decision's mechanism end to end on one concrete case and checked where it ends up against the decision's stated purpose.

**Findings**

1. **BLOCKING — D2, D3 (batten): a batten run's commits go to the prime's shared records branch, not to the pair, so neither the pusher nor its restarts are scoped the way the record says.**
   - Case: a batten run goes Run-Shed done → Land running → Land done → Worktree-Teardown running → done.
   - `battenCommitStatusDeps` commits with `location` at the prime. Its doc says "Push runs from the prime, so it covers only the prime's records side". `PushRun(location, location, ...)` pushes the prime.
   - So the run's "own pair" in D2 is not what a batten pusher pushes. It pushes the hub's shared prime records branch, which every run's batten transitions share.
   - D3's dobby restart only checks "a pair whose local records branch is ahead", so it never sees the prime.
   - D3's teardown clause doesn't cover it either: the archive push carries the pair's weft branch, not the prime's records. Teardown also ends the run's processes, pusher included.
   - End state: Land's and Worktree-Teardown's transitions, and the final done, sit unpushed in the prime records. Nothing restarts a push until some unrelated prime push happens. That misses D3's purpose, "nothing stays unpushed".
   - Fix: say that a batten run's pusher pushes the prime records, make one prime pusher per hub, and add a prime check to the dobby tick and the end of teardown.

2. **MEDIUM — D2: there is a lost-wakeup race between the pusher deciding to exit and the seam checking whether it is alive.**
   - Case: the pusher finds origin at the local tip and decides to exit. A commit lands before the process is gone. The seam's live-process read sees it alive and starts no pusher. The pusher exits.
   - End state: the commit is unpushed until the next commit, the dobby's tick (loom pairs only, see finding 1) or a resume.
   - D2 needs an exit protocol, for example: release a pid lock first, then re-check the tip; or have the seam signal the pusher.

3. **MEDIUM — D4, D1: the "local pause gate" the record leans on is the hub's network-gate admission, not a run pause.**
   - `statuscommit`'s Admit is `pauserecord.Admit`. `hold.go` documents it as the hub's network gate: it "waits at the gate and marks op in flight".
   - D1 says the transition "takes no network-gate admission; it keeps the local pause gate". Those are the same gate, so the sentence contradicts itself.
   - D4 says the pusher pauses the run "through the run's pause intent, which the run's local pause gate meets at its next transition". Nothing in that gate reads a run's pause intent.
   - Today the stop lives in loom's pre-push check (`internal/loomcli/prepush.go`, `errPushStopped`: "the run is paused or held, and the orch was noticed"), and it runs inside the driver's Commit.
   - Case: origin ends the entry. The pusher's reconcile returns "pause", it writes something, and the driver's next transition calls the Commit seam, which no longer runs the pre-push check. Nothing stops the driver.
   - D4 has to name the mechanism the driver checks at its transition: a local read of a stop mark the pusher writes, or the existing pause record that `lym loom pause` writes.

4. **MEDIUM — D5: the pusher calling `pairteardown.HandOver` kills itself, and for batten it also removes the pair mid-run.**
   - `HandOver` first calls `EndSubjectProcesses(prime, slug, false)`. D2 registers the pusher under the slug in runproc, so the pusher ends itself before it archives the tips.
   - D5 does say "ends every process of the run but the pusher itself". It does not say this requires changing `EndSubjectProcesses`/`HandOver`. Without that change the end state is no archive tags and the pair left standing, which misses D5's purpose.
   - For a batten pusher (from the prime, not `InsidePair`), the hand-over also runs `RemovePair`. That is consistent, but it depends on the process end having succeeded first.

5. **LOW — D2/D5: one slug ends up with two pushers.**
   - The batten run (prime records) and its inner loom run (pair records and code) are two runs for the same slug. D2 gives each one pusher.
   - Both can meet the takeover and both call `HandOver`. Only the hand-over mark (`ErrHandedOver`) stops the second, and that mark is written in `HandOverTips` after `EndSubjectProcesses` has already run. If the first pusher is mid-hand-over, the second would end its processes.
   - The record does not discuss two pushers per slug.

6. **LOW — Scope "Out", board writes: holds, worth one line in Constraints.** Every loom transition calls `SetBoardStatus` before the core. I checked whether it waits on the network: `boardCriticalSection` saves under a local lock and "spawns a detached sync", so it does not wait. The record lists board writes as out of scope without saying they don't wait, which is the fact D1's purpose rests on.

**Per decision: case used, purpose met or not**

- **D1**: a skipped loom Drift row, then Run-Shed done → Land in batten. The commit-only path holds; no push or reconcile in the seam. Purpose met, apart from the gate wording contradiction in finding 3.
- **D2**: three skipped loom rows in a burst. One pusher, latest tip, records then code, all hold. Not met: the exit race (finding 2), and batten's scope is wrong (finding 1).
- **D3**: an owed push, then a refused push, then a crash and resume, then teardown. Met for loom pairs. Not met for batten's prime-record transitions after Run-Shed (finding 1).
- **D4**: origin ends the entry mid-run. Not met: the pause has no carrier the driver reads (finding 3).
- **D5**: B adopts after two unpushed loom transitions on A; A's pusher meets the rejection and runs `PushRun` → `HandOver`. Met only if `HandOver`'s process end spares the caller (finding 4). There is a double-pusher edge (finding 5).
- **D6**: an origin outage during a darn run. The run continues and the pusher retries. Met.

**Tool calls**

- `internal/fabricengine/spawn.go` and `internal/pairteardown/handover.go` spans: to check `SpawnStatusPush`'s scope and whether `HandOver` ends every process of the slug. My context had only the subagent's summary.
- `internal/battencli/commitstatus.go` spans: to check where batten commits and pushes (prime or pair). My context had only the summary.
- `internal/loomcli/wiring.go` spans: to check what loom's seam does per transition besides the core (board status, where the reconcile sits). Not in my context.
- `boardengine` `SetRunStatus`: to check whether the per-transition board write pushes synchronously. Not in my context.
- `boardCriticalSection`: to confirm the board write syncs detached. Not in my context.
- `pauserecord.Admit` and the `errPushStopped` sites: to check what the "local pause gate" D1 and D4 rely on is. Not in my context.
