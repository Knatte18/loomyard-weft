```yaml
lens: existing-paths
outcome: reported
```

Lens: existing-paths. I traced the existing code paths that also write or read the state each decision introduces or changes.

**Paths traced, per decision**

- D7, the hub entry the hold writes in the pause record, against `pauserecord`'s one live entry per scope (`internal/pauserecord/doc.go`, "at most one live entry per scope"): **breaks**. It collides with the operator's hub-wide `lym tender pause` and with hibernate's hub entry.
- D7's hub entry against the hibernate wake's reconcile (`internal/hibernateengine/wake.go:58-64`): **breaks**. A hub entry from another caller makes the wake skip its origin reconcile and tell the operator to run `lym tender resume`.
- D7's hub entry against `lym tender resume` with no slug (`internal/tenderengine/resume.go:422-440`): **breaks**. It lifts the hub entry, so the hold's local enforcement goes while the Bolt hold still stands. The record says nothing about whether it is rewritten.
- D7's hub entry against the orch watcher (`internal/orchengine/pause.go`, header and `recordOrchInEntry`): **breaks** D8's "orch is up during a hold". The watcher holds the cycle machine and interrupts a busy orch at the grace end of any live hub entry.
- D7's hub entry against the fix task's own run processes under `adopt-fix`: **breaks**. A hub entry covers every slug, so the network gate and the turn interruption hold the one task the act rule lets act.
- D7's hub entry against the parent-lock exemption (`pauserecord/doc.go`: "A caller whose run holds a parent lock is exempt from slug entries, so only a hub entry holds it"): **breaks**. A run holding a parent lock, such as a Finalize past its parent-side merge, is held on every machine that reads the hold. The parent lock stays taken.
- D7's hub entry against `pairteardown`'s pause reading (`internal/pairteardown/skip.go:139-155`, `pausedIn`): **survives, untested**. While a hold stands, every teardown, D11's included, runs under `hubEntry=true`, which changes its run-lock pass rule. The record is silent; no finding.
- D7's retirement of the `owner` and `adopt` callers, against their writers and readers (`internal/adoptcli/pause.go`, `runreport.go`, `internal/beamcli/dobbydeps.go`, `internal/dobbyengine/passes.go`, `internal/hibernateengine/wake.go:113`, `internal/tendercli/deps.go`): **survives**. D7 names every writer. The readers `beamcli/dobbydeps.go` and `adoptcli/runreport.go` fall under "every writer of them" only as readers. No finding.
- D3, the cancel marker moving into the batten record, against its readers: **survives**. The readers are `pairteardown/teardown.go:196,725` (`cancelMarked`), `battencli/startguard.go:254` (start, step and goto refusal), `dobbyengine/passes.go:124,203` (no pass), `dobbyengine/waitreminder.go:72`, `runnotice.go`, `ended.go`, `battencli/owner.go`, `arm.go` and `dobbyreads.go`. D1 says every reader takes the state from `runfsm`, and D11 names `ended.go`. Each reader needs the intent read from the tracked record; that is enumeration, not a design gap.
- D3, the cancel intent read for D1's `cancelled` against `cancelling`: **breaks**. See finding 7.
- D2, the retirement of the detached push spawn, against the other callers of `fabricengine.SpawnStatusPush`: **breaks**. The callers are `fabricengine/teardownlock.go:169` (the push deferred to a teardown lock's release), `fabricengine/anchorchange.go:557`, `tendercli/deps.go:148` (tender's prime push) and `fabriccli/spawn.go:14`. D2 retires only "the per-transition push behind `statuscommit`" and fabric's pause deferral.
- D2's synchronous push against `fabricengine/pendingflags.go:50` (`pushDeferral`): **survives**. D2 retires the pause deferral by name.
- D4, the takeover read "from the replay's result", against `fabricengine/replay.go:45-48` (`ErrReplayConflict`, "the starting tip is restored"): **breaks**. See finding 8.
- D10, the session id moving out of `orchengine.State`, against its readers (`orchcli/start.go`, `orchcli/status.go`, `orchroute/live.go`, `battencli/overvieworch.go`): **survives**. `orchengine/state.go:471-472` says the caller sets `SessionID`, so the readers move with its one writer.
- D8's retirement of `lym adopt worker` and the adopt record, against fronttask's on-land action (`internal/fronttask/fronttask.go:32-37`, `adopt check &lt;subject&gt;`): **survives**. `lym adopt check` is not on D8's Retired list. The dobby keeps running it on the fix task's landing, which D8 relies on.

**Findings**

1. **BLOCKING, design, D7 and D8 (`adopt-fix`):** the hub pause entry D7 writes on reading a hold covers every slug, so it holds the fix task's own processes, the one run the act rule answers `act` for.
   - D7: "a hub entry in its own pause record, so the network gate and the turn interruption hold its runs' processes".
   - D7: "Under `adopt-fix`, the hold's fix task alone gets `act`."
   - Evidence: `internal/pauserecord/doc.go` ("A hub entry covers every slug").

2. **BLOCKING, design, D7 and D8 (orch up during a hold):** the orch watcher holds the cycle machine and interrupts a busy orch at the grace end of any live hub entry. So D7's hub entry stops the orch that D8 keeps up "to read the fix task's halts and notices".
   - Evidence: `internal/orchengine/pause.go`, the file header and `recordOrchInEntry` at lines 225-240.

3. **MEDIUM, design, D7 against the operator's hub-wide pause and hibernate:** the pause record holds at most one live entry per scope, so D7's hold-derived hub entry shares the `ScopeHub` slot with the operator's `lym tender pause` and hibernate's hub pause.
   - D7 "removes that entry once it reads no hold", which can lift the operator's own hub pause.
   - The operator's `lym tender resume` lifts D7's entry while the Bolt hold stands.
   - The record says neither which caller the entry carries nor how it coexists.
   - Evidence: `internal/pauserecord/doc.go` ("at most one live entry per scope"); `internal/tenderengine/resume.go:422-440`.

4. **MEDIUM, design, D7 against the hibernate wake:** a live hub entry from any other caller makes the wake skip its origin reconcile. Its notice then tells the operator to lift the entry with `lym tender resume`, which removes the hold's enforcement.
   - Under a hold, every wake therefore skips its reconcile, and its notice points at lifting the hold's entry.
   - Evidence: `internal/hibernateengine/wake.go:58-64`.

5. **MEDIUM, design, D7 against a Finalize past its parent-side merge:** a hub entry is the one entry that holds a run holding a parent lock.
   - On every machine that reads a hold, a landing past its parent-side merge is held with its parent lock taken.
   - D1, D6 and D9 carve that landing out of `hand_over` only, not out of a hold's hub entry, so "a landing in flight is never stranded" fails for as long as the hold stands. Under `adopt-fix` at the try limit, that is indefinitely.
   - Evidence: `internal/pauserecord/doc.go` ("A caller whose run holds a parent lock is exempt from slug entries, so only a hub entry holds it"); `pattern/PATTERN-owner-machine.md:23-24`.

6. **MEDIUM, design, D2 (other detached push paths):** D2 retires only the per-transition spawn behind `statuscommit`. `SpawnStatusPush` also serves a push deferred to a teardown lock's release, an anchor change and tender's prime push, and each stays detached.
   - A transition whose push is deferred behind a held teardown lock still lets the run act before origin has the records, against "a transition pushes its records before the run acts".
   - Evidence: `internal/fabricengine/teardownlock.go:169`, `internal/fabricengine/anchorchange.go:557`, `internal/tendercli/deps.go:148`.

7. **MEDIUM, consistency, D1 and D3 (`cancelled` against `cancelling`):** D1 reads `cancelled` from "a finished cancel (D3)", but D3's cancel intent records only its start, machine and remove or keep. No decision writes a finished mark.
   - Today's readers tell the two apart only by the marker's presence and its writer's liveness (`battencli/startguard.go:31-32`, `Marker` reads "the liveness of its writer"), and liveness is machine-local.
   - So another machine cannot derive `cancelled` from tracked records, against D1's acceptance criterion with every `.lym` cache absent.
   - Evidence: `internal/battencli/startguard.go:31-32,254`.

8. **BLOCKING, design, D4 (detecting the takeover on a rejected push):** D4 reads the takeover entry "from the result" of the records replay. The adopter's takeover edits the same batten and child record files the old owner's transition commit edits.
   - Today's replay stops on a conflicting commit with `ErrReplayConflict` and restores the starting tip, which leaves no result to read.
   - The old owner then never reads the takeover, never hands over, and keeps its processes. D4's first acceptance criterion fails in the common case.
   - Reading the record from the fetched origin tip, before any replay, avoids this.
   - Evidence: `internal/fabricengine/replay.go:45-48,425`.

**Tool calls**

1. A grep of the cancel marker's readers, of the users of the `OwnerCaller`, `AdoptCaller` and `HoldCaller` callers, and of `ScopeHub`: my context had their definitions, not their users.
2. Reads of the cancel lines in `pairteardown/teardown.go`, `startguard.go`, `passes.go` and `waitreminder.go`, and of the hub-scope lines in `hibernateengine/wake.go`, `tenderengine/resume.go`, `orchengine/pause.go` and `pairteardown/skip.go`: these confirm how each reader uses the state D3 and D7 change.
3. Reads of `hibernateengine/wake.go:50-75` and `tenderengine/resume.go:415-440`, and a grep of `orchengine.State`'s session-id readers: the wake's and resume's handling of a hub entry, and the readers D10's move meets.
4. Reads of `pairteardown/skip.go:135-165`, of the `SpawnStatusPush` callers and of fabric's push deferral: the teardown's hub-entry read and the detached push paths D2 meets.
5. A read of `orchengine/pause.go`, its header and lines 220-240: whether a hub entry holds the orch, which D8 keeps up during a hold.
