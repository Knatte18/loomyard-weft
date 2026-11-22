```yaml
lens: existing-paths
outcome: reported
```

Lens: existing-paths. I traced the code paths that already write, read, reapply or end each piece of state the decisions introduce or change.

**Paths traced, per decision**

- D1, batten seam `push_pending` marker (`internal/battencli/commitstatus.go:199-289`): survives. The start-row branch retires, and the marker stays tolerant per the Constraint.
- D1, loom `guardedCommit` and `guardedPush` (`internal/loomcli/wiring.go:225-237`): survives. The check leaves the seam, as D7 says.
- D3, `runproc` owner `HubOwner`, `RolePush`:
  - Survives the pair teardown's `EndSparing(slug)` (`internal/pairteardown/teardown.go:761`), because the owner key differs.
  - Survives `HandOver`'s `EndSubjectProcesses(prime, slug)` (`internal/pairteardown/handover.go:103`), and a root `EndSubject` (`internal/runproc/end.go:100`), which spares `RolePush`.
  - Survives the anchor's `EndDobbies`, which spares `RolePush` (`internal/fabriccli/anchor.go:468`).
  - Survives tender `Reap`, which never ends a hub-owner process (`internal/tenderengine/reap.go:35`).
  - Survives harden reap, which ends sandbox hubs only.
- D3/D7, the pause start condition:
  - The pusher's pause check reads `pauserecord` coverage (`pauseCovered`, `internal/loomcli/landingdeps.go:692`; `runread` `pauseCovers`).
  - The pre-push check pauses through `tendercli.PausePair` → `tenderengine.Pause` → `pauserecord.Update` (`internal/loomcli/prepush.go:77`, `internal/tenderengine/pause.go:142`).
  - `lym loom pause` writes the run status's pause conditions instead (`internal/loomcli/cli.go:430-440`, `internal/shedverbs/pause.go:39`). **Breaks** as D7 words it.
- D3/D7, a hub-scope tender pause: it covers every slug (`internal/tenderengine/pause.go:120-123`), so every pair pusher stops. **Breaks** D1's purpose, and the record is silent.
- D5/D8, other writers of owed-class records with `Runs` naming the slug:
  - They are tendercli's `SpawnStatusPush` (`internal/tendercli/deps.go:147`), the anchor change (`internal/fabricengine/anchorchange.go:557`), followup (`internal/pairteardown/followup.go:91`), `lym cleanup` (`internal/battencli/cleanup.go:89`), the code-tip `PushTip` and `GiveUp` (`internal/battencli/codetipsettle.go:62-80`), `HandOverOwnedElsewhere` (`internal/pairteardown/handover.go:183-201`), and Worktree-Teardown's own D5 push.
  - The path: `statusPushRequest` sets `TeardownSlug` (`internal/fabricengine/spawn.go:165`), which becomes `opts.RunSlug` (`internal/fabricengine/pushanchored.go:176-177`). `recordOwedRecordsPush` then writes `Runs` (`internal/fabricengine/pushanchored.go:378-380`). `FailedPushesOwedBy` reads it back (`internal/fabricengine/failedpush.go:276-292`), and `runread` sets `PushOwed` (`internal/pairteardown/runread.go:231-240`).
  - **Breaks** D8's "the run goes on while its push is owed".
- D5, code-side refused records: `checkOrigin`/`holdBackRewound` and `failRejection` write records keyed by the code branch (`internal/fabricengine/pushclassified.go:635,716,566`). D5's restart gate reads only the records branch. **Breaks**: the pusher restarts and is refused on every commit.
- D5/D6, the owner's code-tip settle: `settleCodeTip` pushes a held tip only on a `ReasonCodeTip` verdict (`internal/battencli/codetipsettle.go:117,144-150`; called from `internal/battencli/dobbyreads.go:260`). D6 unsets `CodeTipMissing` on the owner machine, and D5's dobby tick reads only the records branch. **Breaks**: a code tip whose pusher died after the records landed has no restart.
- D6, the hand-over mark (`internal/fabricengine/takeover.go:120-125`) and `HandOverTips` (`internal/fabricengine/handover.go:66`): survive, given D6's stated changes.

**Findings**

1. **BLOCKING · design · `_lym/discussion/decision-record.md#D7`**
   - Summary: D7 says the reconcile pauses the run with "the pause record `lym loom pause` writes", but that verb writes pause conditions into the run status (`internal/loomcli/cli.go:430-440`; `internal/shedverbs/pause.go:39`, `applyPauseConditions` on `shedengine.Status`). D3's start condition and today's pre-push pause use `pauserecord` coverage (`internal/loomcli/prepush.go:77` `tendercli.PausePair`; `internal/tenderengine/pause.go:142`; `internal/loomcli/landingdeps.go:692-704`).
   - Effect: a pause written as D7 names it is invisible to D3's check, so the next commit or dobby tick restarts the pusher and pushes past the stop.
   - Evidence: the files and lines above.
2. **BLOCKING · design · `_lym/discussion/decision-record.md#D6`**
   - Summary: D6 unsets `CodeTipMissing` on the owner machine when the local code branch holds the tip. The owner dobby's `settleCodeTip` is the existing path that pushes a held tip, and it acts only on a `ReasonCodeTip` verdict, so it never fires there. D5's dobby tick restarts a pusher only when the records branch is ahead.
   - Effect: a pusher killed after its records push landed and before its code push has no restart until the next commit or resume. A halted run's code tip stays off origin, and on another machine it ends in the grace give-up. That contradicts D5's "every path that can leave a commit behind has a restart".
   - Evidence: `internal/battencli/codetipsettle.go:117,144-150`; `internal/battencli/dobbyreads.go:260`; `internal/pairteardown/runread.go:241-250`.
3. **MEDIUM · design · `_lym/discussion/decision-record.md#D8`**
   - Summary: D8 says the run's push-owed input stays unset while the pusher retries, but existing pushes still write owed-class records naming the run's slug: the detached `SpawnStatusPush` callers, the code-tip settle, `HandOverOwnedElsewhere`, and Worktree-Teardown's own D5 push, whose mode D5 leaves unstated.
   - Effect: during an outage, `ReadRun` sets `PushOwed` and the act rule holds the run that D8 says goes on. Only the pusher's own records are exempt.
   - Evidence: `internal/fabricengine/spawn.go:165`; `internal/fabricengine/pushanchored.go:176-177,378-380`; `internal/fabricengine/failedpush.go:276-292`; `internal/pairteardown/runread.go:231-240`.
4. **MEDIUM · design · `_lym/discussion/decision-record.md#D7`**
   - Summary: the pusher's pause gate stops for any `pauserecord` coverage, including a hub-scope tender pause, which covers every slug (`internal/tenderengine/pause.go:120-123`). Today `skipsCheck` lets a paused run's push through (`internal/loomcli/prepush.go:99-100`).
   - Effect: every paused pair, the paused transition included, stays off origin for the whole pause, and the record states no bound for it.
   - Evidence: the files and lines above.
5. **MEDIUM · design · `_lym/discussion/decision-record.md#D5`**
   - Summary: `pushClassified` writes refusal-class records for a pair's code side, keyed by the code branch (rewound, gone, not_fast_forward, other), but D5's restart gate checks only the records branch.
   - Effect: a refused code side restarts the pusher on every commit, and each pass is refused again — the loop round 1's F1 closed for the records side.
   - Evidence: `internal/fabricengine/pushclassified.go:566,635,716`.

**Tool calls**

1. `internal/tenderengine/reap.go` 1–58 and the orphan grep: the manifest lacked Reap's rule on which owners it ends.
2. A grep of `internal/loomcli/cli.go` and `internal/shedverbs` for pause: the manifest lacked what `lym loom pause` writes.
3. `internal/fabriccli/anchordeps.go` 455–475 and the `EndDobbies` caller: the manifest lacked that end path's spare list.
