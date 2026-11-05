```yaml
lens: existing-paths
outcome: reported
```

Lens: existing-paths. I traced what D2, D3, D7, D9, D10 and D12 meet in code that already runs.

**Paths traced, per decision**
- D2, promote with recipe `raddle` against `endsAsSketch` and `checkRunLocks`: survives. `IsSketch` needs kind note, so a raddle task is no Sketch (`internal/boardengine/sketch.go:35`).
- D2/D10, the finish's `MakeSketch` (`internal/loomcli/wiring.go:234`): survives. D10 skips it by rows, and `MakeSketch` already skips a done entry (`sketch.go:77`).
- D3, Record-Write resume (`recordwrite.go:114`, an existing file is never overwritten): survives. A seeded record is kept on resume.
- D3 against `seedDiscussedRecord`, the Motif gate: survives. Both read `Board.DecisionRecord`.
- D7, the run_settings notice saying a running pair takes changes through `--layer local` (`runsettings.go:79`): survives. The mode is fixed at the first spawn.
- D9 step 3, the run lock on the run's own entry going done (`scopeChanged`, `runlock.go:193`): survives.
- D9 step 3, later `SetRunStatus` (`board.go:203`) and `Prune` with `notify_on_land` cleared (`store.go:793`): survive.
- D9 step 3, batten resume reconcile (`resumereconcile.go:114-124`): **breaks** (F1).
- D9 step 3, hub reconcile and `lym cleanup` teardown of an ended entry (`originreconcile.go:301-311`, `pairteardown/originseams.go:155`): **breaks** (F1).
- D9 step 3, weft replay in `fab.Pull`, which Land itself calls (`land.go:96-103`, `pull.go:412-422`, `replay.go:305-320`): **breaks** (F1).
- D9 step 3, loom pre-push (`loomcli/prepush.go:206-215`): passes only when the parent holds the task's work; otherwise it stops the push (F1).
- D10, the raddle store in the board commit against the Bolt replay (`boardengine/replay.go:44-53`, `fabricengine/replay.go` replayBolt): **breaks** (F2).
- D12, `Result.Usage` readers (`shedadapters/roundusage.go`) against `SessionUsage` gaining fields: the fields survive. The record does not say what becomes of the existing token writer (F3).
- D12, the usage read at `wait.go:1931`, done-only today: D12 states the change, so nothing is undone.

**Findings**

F1 — BLOCKING, design, D9 step 3 and its Bound (also the D9 acceptance criteria on retry after step 3).
- Summary: marking the run's own entry done at step 3, before steps 4–5 and the run's end, makes every origin-reconcile path read the live run as ended. A halted or crashed Raddle-File cannot be resumed to finish the beam and the notice.
- The ledger marks a run ended when its entry is done (`internal/boardengine/replay.go:272`, `Ended: isDone(task)`).
- `resumeRefusal` then refuses any batten resume: "origin ended … run lym cleanup --apply … instead of resuming it" (`internal/battencli/resumereconcile.go:120-124`).
- The hub reconcile lists the entry as ended and tears its pair down (`internal/fabricengine/originreconcile.go:301-311`, `internal/pairteardown/originseams.go:155`).
- The weft replay sets aside the run's local record commits, because a raddle run never enters a Finalize row, so `withoutFinalizeEntered` leaves it in the ended set (`internal/fabricengine/replay.go:305-320`, `finalizeEnteredRead` at :236-251). That replay runs in `fab.Pull`, which Land itself calls (`internal/battencli/land.go:96-103`).
- The record's Facts cover only the run lock, `SetRunStatus`, `MakeSketch` and `Prune`, and say nothing of these paths.
- So the criterion "a Raddle-File interrupted after any step and run again finishes the remaining steps", and the halt-after-step-3 handling, cannot hold.

F2 — MEDIUM, design, D10 and D9 step 3 ("the staged store is written into the raddle store in the same write").
- Summary: the board's Bolt replayer claims only `board.json`, the overviews and `design-`/`decision-` files at the Bolt root. A commit that also holds the raddle store directory is replayed by cherry-pick, not entry by entry.
- When another machine wrote the board meanwhile, step 3's commit then conflicts on `board.json` instead of merging by entry.
- Evidence: `FabricReplay.Claims` and `boardRecordSlug` refuse any path with a `/` (`internal/boardengine/replay.go:44-53, 70-78`). replayBolt sends a commit with an unclaimed path to cherry-pick, and a conflict returns `ErrReplayConflict` (`internal/fabricengine/replay.go`, replayBolt doc and line ~675).
- The record is silent on how the store's files replay, or which replayer claims them.

F3 — MEDIUM, consistency, D12 against the declared `PATTERN-run-usage-record`.
- Summary: `internal/shedadapters/roundusage.go` already records each review half's token reading (header and the `roundUsage` Review/Fix/Seed/Judge `halfUsage`).
- D12 adds "one writer in shed's shared machinery", and the declared PATTERN says "only shed's one usage writer". The record neither folds the round usage writer into the new one nor exempts it, so the existing path becomes a second token writer that contradicts the declared entry.
- The Constraints Fact at `decision-record.md:197` cites that file only as evidence that nothing records cost.

Tool calls:
1. grep of `boardCriticalSection` for commit staging — to see which files a board write commits (no hits).
2. `land.go:95-140` and the `MakeSketch` callers — to find Land's pull and prune seams and the end-as-Sketch finish.
3. grep of the `replayWeft`/`weftEndedByOrigin` callers and `RefuseWhenBusy` — to find when the weft replay runs.
4. grep of `DecisionFileName`/`decisionFilePrefix` — to find how decision files reach the board commit and replay.
5. `boardengine/replay.go:20-110` — the Bolt replayer's claimed paths.
6. replayBolt's lines on claims — how a commit with an unclaimed path is replayed.
