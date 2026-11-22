```yaml
lens: existing-paths
outcome: reported
```

Lens: existing-paths. I traced each state D1–D8 introduce or change through the code paths that already write, read or reset it.

**Paths traced, per decision**

- D1, network-gate admission dropped from the transition: survives, because `PushPairAnchored` takes its own `pauserecord.Admit` (pushanchored.go:158), so the pusher still passes the gate.
- D1, `push_pending` and start-row retirement: survives. The marker reader is batten's seam alone (commitstatus.go:278).
- D1, the run's other synchronous pushes: **breaks.** Batten's `commitRunRecords` (drive.go:297, 348) and loom's `commit-records` verb (commitrecords.go:59) still push synchronously and still run the pre-push check. The record is silent on them.
- D3, pusher registered "under its worktree, not under a run's slug": **breaks.** A pair's worktree name is its slug, and `SpawnStatusPush`'s child registers under `TeardownSlug`.
- D3, the other pushers of the same worktree: survives. The push lock serialises them. These are catch-up's `pushOwedForCatchUp`, the code-tip settle's `PushTip`, the commit-spawned detached push and cleanup's `spawnStatusPush`.
- D4, transition commit takes the weft write lock: already holds today (commitweftpaths.go:53).
- D5/D8, owed retries write no failed-push record, so `ReadRun` reads no `PushOwed`: **breaks.** `PushPairAnchored` writes Deferred, Transport and TeardownLock records, and `FailedPushesOwedBy` feeds them into `PushOwed`.
- D5, refused-push record stops the pusher "for the worktree": **breaks for the prime.** The record is keyed per repo and branch, so one refusal stops every batten run's pushes.
- D6/D8, code-tip window: **breaks.** Records land first and code later, while the run goes on. In between, `ReadRun` sets `CodeTipMissing` and the act rule makes the run wait.
- D7, pause after a reconcile stop: **breaks.** `skipsCheck` skips the check once a pause covers the run, so the pusher's next pass pushes unchecked.
- D5, Worktree-Teardown push, then exclusive teardown lock: survives. The push takes the lock shared and the removal takes it exclusive, in order.
- `tender resume` drops `PushOwed` (`resumeAside`): survives.

**Findings**

1. **BLOCKING, design. D5 and D8: an owed retry still writes a failed-push record, so the act rule holds the run.**
   - The pusher pushes through `pairteardown.PushRun` and `PushPairAnchored` (D3).
   - `PushPairAnchored` writes a record of class `PushFailureDeferred` (pushanchored.go:365-385), `PushFailureTeardownLock` (pushanchored.go:296-318) and, per its doc, `PushFailureTransport` (pushanchored.go:137-139).
   - `FailedPushesOwedBy` returns exactly those classes for the run (failedpush.go:276-291). `ReadRun` turns them into `PushOwed` (runread.go:231-237), and the act rule answers wait (runfsm/act.go:119).
   - Catch-up's `pushOwedForCatchUp` writes the same records (catchup.go:260-270).
   - So D8's "the act rule's readers keep their behaviour" and acceptance criterion "D8: the dobby still schedules it" fail unless fabric's owed recording changes. The record names no such change. D3's bound ("only the sides `PushPairAnchored` already pushes") implies fabric is unchanged.

2. **BLOCKING, design. D3: "registers under its worktree, not under a run's slug" does not keep a pair's pusher apart from the run.**
   - A pair's worktree name is its slug: `statusPushRequest` sets `TeardownSlug = l.WorktreeName`, and the detached push registers under that owner (spawn.go:124-127, 160-166).
   - `runproc.End` and `EndSubject` by slug therefore end the pusher. Teardown, cancel and `HandOver` → `EndSubjectProcesses` (endsubject.go:25-40) all do this.
   - That breaks acceptance criterion "D3: ending a run's processes leaves its worktree's pusher running". The record names no owner key distinct from the slug.

3. **MEDIUM, design. D7: once the pusher's reconcile pauses the run, the existing skip rule makes the next pass push unchecked.**
   - `skipsCheck` returns skip when `pauseCovers` (prepush.go:96-98), and `guard` then lets the push go (prepush.go:131-133).
   - D7 says the pusher "pushes nothing more for that pair until the run resumes", but the reused check does the opposite.
   - D1 lets the run keep committing until the driver honours the pause, so commits do exist to push.

4. **MEDIUM, design. D6 and D8: the existing missing-code-tip wait now holds the run between the pusher's records push and its code push.**
   - Records land first and code only after (pushanchored.go:250-260).
   - Meanwhile `ReadRun` reads `CodeTipMissing` (runread.go:242-247) and the act rule answers wait (runfsm/act.go:123), while the run acts locally.
   - Today that window sits inside the synchronous transition. Under D1 a dobby tick in it reads the run as waiting on a push.
   - D6 says the settle covers this "as they do today", which does not hold once the run no longer waits at the transition.

5. **MEDIUM, scope. D1 and Scope: the run's other record pushes still wait.**
   - Batten's `commitRunRecords` pushes synchronously at driver stops (commitstatus.go:131-150, drive.go:297, 348).
   - Loom's `commit-records` verb pushes synchronously and runs the pre-push check (commitrecords.go:51-75).
   - Neither is a row's own act, and neither is in Scope's In or Out. D1's "no step waits for a push" is broken by both.

6. **MEDIUM, design. D5: a refused push of the prime's records stops the pusher for every batten run in the hub.**
   - Failed-push records are keyed by repo and branch (failedpush.go:158-159). The prime's records branch is shared by every batten run (D3).
   - So one refusal leaves every batten run's commits local until the record is cleared. D5's bound states this per worktree, without the hub-wide effect.

7. **LOW, consistency. D4 states as new a lock the commit already takes.**
   - `CommitRecordsPaths` takes the weft write lock blocking (commitweftpaths.go:49-57). Both seams commit through `CommitAnchoredPathsWithTrailers` → `CommitAnchoredPaths` (trailercommit.go:92).
   - `retryWeftPush` try-acquires the same lock (pushclassified.go:835-841).
   - D4 and its acceptance criterion describe existing behaviour as a change, and its rationale ("without the lock …") reads as if the guard were missing.

8. **LOW, consistency. "Notes for the plan writer" calls `spawnStatusPush` unused, but cleanup uses it.**
   - cleanup.go:89 calls it: `Cleanup` spawns the prime's records push through it.

Tool calls:
1. Grepped `owedPush` in pairteardown and `commitRunRecords` callers. The context lacked how `PushOwed` is filled and where batten's records commit is called.
2. Read runread.go:140-150 and grepped the owed-push lookup. The context lacked `owedPush`'s source.
3. Read `FailedPushesOwedBy` and grepped the clear paths. The context lacked which record classes count as owed.
4. Read codetipsettle.go's head and `CodeTipMissing` in the act rule. The context lacked how a missing code tip affects the run.
