```yaml
lens: existing-paths
outcome: reported
```

Lens existing-paths: for each decision's new state, trace the existing code paths that write or read it, and say whether the decision survives each one.

**Paths traced, per decision**

- D1, a board write's `spawnSync` (`boardengine/board.go:217`, which `SpawnSync` wraps) moves to the pusher: survives.
- D1, `Fabric.commit`'s detached push through `spawnDetachedPushFn` (`fabricengine/commit.go:171`), which D1 covers: survives.
- D1/D5, synchronous pair pushes outside the pusher, against D5's pair-target hold:
  - the catch-up's `pushOwedForCatchUp` (`battencli/catchup.go:265-275`): breaks.
  - PR-Review's `pullRequestRecordCommitter` (`loomcli/landingdeps.go:473`): breaks.
  - cancel's `PushRecords` (`battencli/cancelcmd.go:365`): breaks.
- D2, the teardown's `PushUntilLanded` (`battencli/wire.go:1802`), which D13 removes: survives.
- D3, the pusher's `RefusedPushStands` start and loop conditions (`runpusher/start.go:122`, `run.go:66`), next to the ledger's refused mark: survives as written. The mark copies that state.
- D5, batten's Land row prime pull (`battencli/land.go` `Pull` → `Fabric.Pull` → `pullWarpArm`): breaks.
- D5, batten's synchronous status push of the prime's records, which carries the Land and teardown transitions (`battencli/commitstatus.go:182-184`): breaks "origin never reads".
- D5, `pushParent`'s rejected-push repair through `archiveHeldParentMerge` and `adopterFirstRepair` (`landingshed/finalize.go:974-985`, `816-836`), which moves to the pusher: breaks, since the record is silent about it.
- D5, `markTaskDone` through `SetStatusPushed`, a synchronous pull and push (`loomcli/landingdeps.go:238-244`, `boardengine/board.go:244-274`), made by the parent target's pusher: survives. A failure there is retried on the next pass.
- D5, the run reader's `battenAtLanding` → `PastParentMerge` (`pairteardown/runread.go:270-271`): survives here. It misreads on another machine through the batten record (see F-EP2).
- D6, the step-side halt note (`loomcli/halt.go` `loomAfterStep`): the record addresses it, so it survives.
- D6, the teardown's and cancel's driver-lock reads (`pairteardown/teardown.go:783`, `925`; `battencli/cancelcmd.go:197`) against a driver that holds `DriverLock` while it waits: survives. Teardown waits, and cancel stops the wait.
- D7, the moves of an origin branch in the hand-over (`fabricengine/handover.go:274`) and destroy (`destroy.go:2697`): survives, as the record states.
- D9, `mergeParent` writes the merge record only after the after-merge clean check (`landingshed/finalize.go:1360` vs `1388`). A resume reads the previous merge's record: survives only in part (F-EP5).

**Findings**

- **F-EP1 — BLOCKING, design, `_lym/discussion/decision-record.md:111` (D5).** After a local landing on the prime's branch whose push is still owed, batten's Land row pulls the prime and stops Stuck. The record never mentions the Land row.
  - Evidence: `Pull` calls `fab.Pull` (`battencli/land.go`, `landDeps` `Pull`).
  - `pullWarpArm` fetches first, so an unreachable origin fails there (`fabricengine/pull.go:565-566`).
  - With origin reachable, a local `main` ahead of `origin/main` is neither equal nor a fast-forward of origin. It goes to `refuseUnsafeRewrite` (`pull.go:593-614`), which "refuses what a reset would lose … A local commit no reflog value reaches refuses with ErrWarpDivergedUnpushed" (`pull.go:617-621`). The tracking ref's reflog never held the local landing commit.
  - The Land row then returns Stuck "pull failed: …" (`battenshed/land.go:269-270`).
  - So every local landing whose parent push is still owed halts the batten run at Land. That contradicts the Goal ("only a refusal that retrying cannot fix stops anything") and D5's done claim.
  - Fix: in D5, say what Land's pull does while the parent target owes a parent push. For example, Land skips the code-side pull, or reads a local `main` ahead by only owed tips as level. Add a D5 criterion: a batten run whose landing push is owed passes Land.
- **F-EP2 — MEDIUM, design, `_lym/discussion/decision-record.md:135` (D5's "origin never reads the run's records … whose parent lacks the run's work").** That promise holds back only the pair target. The prime's records still reach origin early:
  - Batten's status commits push them synchronously (`battencli/commitstatus.go:182-184`), and the prime's pusher pushes them too.
  - They carry batten's Land, Worktree-Teardown and done transitions before the parent push lands.
  - On another machine, the run reader reads a batten record at Land or Worktree-Teardown as `PastParentMerge = true` (`pairteardown/runread.go:207`, `270-271`), while origin's parent lacks the work.
  - Fix: in D5, either hold the prime's records past the run's Land transition while the landing entry stands, or narrow the claim to the pair's records and state what a reader of origin's batten record reads meanwhile.
- **F-EP3 — MEDIUM, design, `_lym/discussion/decision-record.md:133` (D5 pair-target hold).** "The pair target pushes nothing past [the records commit] while the entry stands" binds only the pusher. Synchronous `PushRun`s of the same pair bypass it:
  - The dobby's catch-up pushes the pair whenever it is ahead (`battencli/catchup.go:251-257`, `265-275`), so a dobby restart pushes Finalize's records commit before the parent.
  - `pullRequestRecordCommitter` (`loomcli/landingdeps.go:473`) and cancel's `PushRecords` (`battencli/cancelcmd.go:365`) do the same.
  - Fix: in D5, state that the hold binds every push of the pair's records side, the synchronous ones D1 and D8 keep included, or say for each of them why it cannot run while the landing entry stands.
- **F-EP4 — MEDIUM, design, `_lym/discussion/decision-record.md:136` (D5 settle rule).** Today's `pushParent` handles a rejected parent push for which origin's parent already holds the task's work: it archives the local merge through `ArchiveParentMerge`, or adds the adopter-first repair (`landingshed/finalize.go:974-985`, `816-836`, `finalize_owner.go:179-208`). After D5 the parent push is the pusher's own fast-forward-only push.
  - D5 settles an entry only when origin's parent holds "its owed tip". In the adopter-first case origin holds the work but not this machine's squash tip, so the push is rejected, never settles, and the archive and repair paths are dropped with nothing said.
  - Fix: in D5, state what the parent target does with a rejected push while origin's parent already holds the task's work: it archives the local merge as today and settles the entry, or names the adopter-first repair in its refusal.
- **F-EP5 — LOW, design, `_lym/discussion/decision-record.md:210` (D9).** `mergeParent` writes the merge record only after the after-the-merge-in clean check (`landingshed/finalize.go:1360`, `1388`). A Finalize killed or halted between the merge-in and that write leaves the previous merge's record, or none on a first merge-in. A resume then reads an older pre-merge head, or no base, and D9's removal does not run for that merge.
  - Fix: in D9, write the merge record before the after-merge clean check, or state that the removal's base on a resume is the record of the merge the resume finds.

**Tool calls**

1. Read `battencli/land.go` `pullStopOutcome` and `fabricengine.Fabric.Pull`, to see the Land row's pull outcome; my context held only the wiring.
2. Read `fabricengine/pull.go` `pullWarpArm`, to see what the prime pull does when local `main` is ahead of origin.
3. Read the `battencli/commitstatus.go` push, the `dobbyengine/runwatch.go` post sites and `pull.go`'s `refuseUnsafeRewrite`, to confirm batten's synchronous prime records push and the refusal of a local-only commit.
