```yaml
lens: existing-paths
outcome: reported
```

Lens: existing-paths. I traced the state each decision adds or changes through the code that already runs.

**Paths traced, per decision**

- D1, `runproc.End(<beam slug>)` from `lyx batten cancel` or an owner pause of the beam's landing run: survives (the bound in D1 states it).
- D1, `lyx reed down` in a beam worktree, through `EndBeforeSessionDown(<worktree name>)` and `runproc.End`: breaks. The record does not mention it.
- D1, tender's orphan reap (`orphanOwner`): survives. A beam slug is a worktree base name, and `HubOwner` is never read as an orphan.
- D1, the pair teardown of a landed beam ending its dobby: survives.
- D3, a run already torn down whose batten status still reads done: survives, since done is not running and gets no pass.
- D4, the InnerRun owner check standing in for the skipped origin reconcile: breaks.
- D4, InnerRun state that lives for one Call only (`childWait`, `lockWatch`): breaks under one check per Call.
- D4, the producer fields that last across Calls (`noticeEpisode`, `failures`, `restep`, `ownedBy`, `ownerSleep` and the others): the record leaves them unnamed (see finding 3).
- D4, the revived marker keyed by pid: survives, because D4 makes the revive per episode.
- D4, the spawn-confirmation marker keyed by pid: survives, because D4 keeps it per pass.
- D4, `holdSpawnWhileCovered`'s wait loop under a pause or an owner hold, plus the Land, row-notice and prime-lock sleeps: breaks the "one check, then exit" shape. A pass stays in flight for the whole hold.
- D4, the batten-watched marker readers (`loomcli/start.go:94`, `drivewiring.go:265`) given dobby's pid: survives.
- D4, `ResumeInWindow` callers in `beamcli/deps.go:115` and `adoptcli/deps.go:92`: survives (D4 covers them).
- D4, the Open-Window re-run on resume: survives (a continue pass skips it).
- D4, pairteardown's claim settlement and `settledForSessionDown`: survive (D4 names them).
- D4, `driverChecksSkipped` in `EndSession`, reached from `lyx fabric remove` and Worktree-Teardown: breaks. It reads "Finalize Done, no live batten" between passes.
- D5, `lyx batten cancel` ClearClaim (`cancelcmd.go:285-305`): survives, since D5 changes it to `draft`.
- D5, pairteardown `settleClaim` → `claimClear` → `writeBoardClaim(nil)` after `lyx fabric remove`: breaks.
- D5, `IsRunStatus` with `draft` → `claimKeep`: survives.
- D5, `board.json` keeping unknown top-level keys (`store.go:66-70`, `Save` at `store.go:283`): survives.
- D8, the loom template's `arbiter_max_spawns: 3`: survives only if the template becomes `~`, which the "tackler overrides' shape" implies.
- D8, arbiter keys set in task `run_settings.loom` and in worktree layers, where darn's copy does not reach: partly breaks (finding 7).
- D10, the board task guard (`isRunAgentSession`) for a tidy agent that only amends: survives.

**Findings**

1. **BLOCKING, design, D5.**
   A pair removed by `lyx fabric remove` while its batten run is halted, or was never seeded, gets its status cleared to empty, and dobby then starts the task fresh.
   D5 sends only `lyx batten cancel` to `draft` and says nothing about the teardown's own claim clear.
   Evidence: `internal/pairteardown/claim.go:45-50` (`claimClear` for unseeded, paused, blocked, failed and awaiting), `claim.go:172` (`board.SetStatus(slug, nil)`), and `internal/pairteardown/teardown.go:393` (`settleBoardClaim` after `remove`).

2. **BLOCKING, design, D4 continue pass.**
   The record says that between reconciles "InnerRun's own owner check covers it", which is false.
   The check fires only on a sleep seen inside the same process, a pause release, or a retry while owned elsewhere.
   A machine suspend between short passes is never seen. Dobby itself survives the suspend, so every pass after wake is a continue pass with no reconcile.
   That breaks `PATTERN-owner-machine`: acting after a wake starts with the origin reconcile.
   Evidence: `internal/battenshed/innerrun_owner.go:67-75` (the `slept || released || retry` trigger), and `innerrun.go:165-170` (`ownerSleep` and `ownerDue` held in memory).

3. **BLOCKING, design, D4 memory record and Open risks.**
   The design rests on InnerRun's memory, and the record leaves that field inventory to Plan.
   Part of that memory is not producer state that could move to a record: it lives in `childWait` and is reset on every Call.
   The worst case is the parent lock's idle watch (`lockWatch.idle`, "across the checks of one Call"). With one check per Call it never builds up, so an idle child's parent lock is never released.
   The same per-Call fields hold `resumeRetryAt`, the busy and not-parked latches, `lastProbe` and `sleep`.
   Evidence: `internal/battenshed/innerrun_parentlock.go:14-18`, and `innerrun.go:554-600` (the `childWait` fields, "the state of one Call's wait").

4. **MEDIUM, design, D4 and D12.**
   `holdSpawnWhileCovered` sleeps inside the Call for as long as a pause covers the slug or another machine owns the run.
   A pass therefore stays in flight for a whole tender pause. D4 then blocks the "pass at once on lift" from D12, and the pass exceeds D4's "one check per Call".
   The Land row's sleeps and the row-notice retry spacing also keep a pass alive, which D4's per-run bound does not state.
   Evidence: `internal/battenshed/innerrun_pause.go:50-71`, `land.go:377` and `land.go:414`, `rownotice.go:215`, and `primelockwait.go` (`primeLockWaitBound`, 10 minutes).

5. **MEDIUM, design, D1.**
   `lyx reed down` in a beam worktree calls `EndBeforeSessionDown(<worktree name>)`, which runs `runproc.End` on the beam's slug.
   That ends the beam's dobby, and nothing brings it back before the next `lyx beam start`.
   D1's bound names only a cancel or an owner pause of the beam's landing run.
   Evidence: `internal/reedcli/up.go:98`, and `internal/pairteardown/skip.go:62-90` (`endOwned`).

6. **LOW, design, D4 pairteardown.**
   D4 rewrites two named readers of a live batten, but the third use of `battenAlive` is `driverChecksSkipped` in `EndSession`.
   Between passes, a `lyx fabric remove` after Finalize Done reads "no live batten", skips the quiet wait and removes the pair while Land is still pending.
   The rule belongs on `battenAlive` itself (a live dobby driving the run counts), not on each reader.
   Evidence: `internal/pairteardown/skip.go:25-50`, and `teardown.go:225`.

7. **LOW, design, D8 migration.**
   The config migration copies only the arbiter keys a hub layer sets in `loom.yaml`.
   Arbiter keys set in a task's `run_settings.loom` or in a worktree's `loom.yaml` keep reaching darn today, and after the move they silently stop.
   Evidence: D8's migration text, and `internal/loomengine/config.go:592-593` (the arbiter keys are `CategoryNone`, so a local layer may hold them).

**Tool calls**

- Grep for callers of `EndBeforeSessionDown`, `settleBoardClaim` and `driverChecksSkipped`: the manifest had the definitions but not the entry points that reach them.
- `teardown.go:215-235` and `380-398`, plus `reedcli/up.go:85-100`: needed to confirm that `fabric remove` and `reed down` reach the claim clear and the end of the beam slug.
