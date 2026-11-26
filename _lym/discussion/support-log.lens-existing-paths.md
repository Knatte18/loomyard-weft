```yaml
lens: existing-paths
outcome: reported
```

Lens: existing-paths. I traced the code paths that also write or read what D1–D6 introduce.

**Paths traced, per decision**
- D1, reading from `RunView` and `ReadChild`: survives. The tick's step gating (anchor change, adopt hold, catch-up) delays a reading but loses none, because D4's memory compares against the last reading told.
- D2, the `run_watch` lookup through `lookupTaskSetting`: breaks.
  - `internal/beamsetting` and `dobbyengine/config.go`'s validation close the per-task key set at three keys.
  - That validation, not a per-tick fallback, is what judges a bad value.
- D2, run_settings reaching the task through batten's copy into the local layer: unstated. The local layer takes only keys classified `CategoryNone`, and the record classifies the gate keys but not `run_watch`.
- D3 against `landnotice.go`'s remembered `Requested`: the two can disagree.
- D4 against `passes.go`'s delete of `s.Runs[slug]`: survives. The memory goes with the entry, as D4 says.
- D4 and D5 against `clearDoneRuns`: breaks for the landed line.
- D6 against `gatecli`'s `Holder{Worktree: location.WorktreePath()}` at each acquire site: survives if matched on a path boundary. A raw prefix match breaks on sibling paths.
- D6 against `Pool.Holders` dropping unlocked records: survives.

**Findings**

1. **MEDIUM, D3/D4/D5.** `clearDoneRuns` can clear a landed run before its watch line posts.
   - Evidence: `internal/dobbyengine/donerun.go` holds a done run back only while the landing-notice memory or the landing count holds something of it.
   - Once its pair is gone, the run is cleared, and `passes.go:335` deletes `s.Runs[slug]`, D4's memory included.
   - A landed watch line still being retried under D5's three attempts is lost, and so is every landed line if the watch step runs after `clearDoneRuns` in the tick.
   - Fix: the record should order the watch before the clear, and the clear should wait on an unposted landed or cancelled line, as it waits on `LandNotices`.

2. **MEDIUM, D2.** The record's handling of a bad `run_watch` value conflicts with the existing validation path.
   - Evidence: `internal/beamsetting/beamsetting.go` declares the per-task settings as "pull_request, land_notice and autonomy".
   - `dobbyengine/config.go:57`, `:100` and `:139` validate only those three, and reject a resolved bad value there.
   - D2's rule ("refuses the dobby's lookup … the run reads as `stops` for that tick, logged once") is a new and different failure path.
   - Fix: D2 should add `run_watch` to `beamsetting`'s key and value declarations and to `config.go`'s per-task check, and keep one failure behaviour.

3. **LOW, D2.** `run_watch`'s config category is not stated.
   - Evidence: `run_settings` reach a task through batten's copy into the local layer, which takes only `CategoryNone` keys (`PATTERN-config-layers`). `config.go:73` classifies the per-task keys for that reason.
   - Fix: D2 should say `run_watch` is `CategoryNone`, as D6 does for the gate keys.

4. **LOW, D3.** D3 skips the landed line by reading `land_notice` again, while the landing notice posts from its own memory.
   - Evidence: `landnotice.go` decides from `Requested`, remembered ahead of Land, and keeps the last reading when a read fails.
   - A fresh `LookupLandNotice` at the landed tick can read differently after a change or a read failure, so the orch gets both lines or neither.
   - Fix: D3 should key the skip off `State.LandNotices`' remembered decision for the run.

5. **LOW, D6.** "Lies under that run's task worktree" needs a path-boundary match.
   - Evidence: each `gatecli` acquire site records `location.WorktreePath()`, and worktrees are siblings such as `<hub>/dobby-run-watch` and `<hub>/dobby-run-watch-weft`.
   - A raw prefix match makes slug `dobby`'s worktree claim holders under `dobby-run-watch`.
   - Fix: D6 should state the match as a path prefix on a separator boundary.

Tool calls:
- Grepped `land_notice` and autonomy readers, and read `donerun.go`: needed the per-task key declarations and the done-run clear ordering.
- Read `RunState` in `state.go` and grepped `Runs` deletions: needed where D4's memory is removed.
- Grepped `gatecli`'s `Holder{Worktree:` acquire sites and weaver worktree creation: needed what path a slot holder records.
