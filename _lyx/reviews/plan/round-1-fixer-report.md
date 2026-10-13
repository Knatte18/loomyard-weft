# Round 1 fixer-report — plan

## Changes

- **F1** (fixed as proposed): card 21 lists `internal/battencli#stepMarkWatched_HoldsTheMarkerOnlyWhileItsNoticesReachTheDriversParent` under `Edit:`.
  A test bullet says its held rows expect `shedrun.BattenWatchedContent(os.Getpid(), BattenDir(h.Location, slug), battenrecipe.NameRunShed)` in place of the pid line.
  It also says the file drops its `strconv` import, whose only use (`lifecycle_integration_test.go:1319`) is that pid line.
- **F2** (fixed as proposed): card 2's refusal-spec sentence adds the `config: tackler not a model-spec` row, worded as the arbiter row is, with the run_settings trigger and the empty-takes-`haiku` way forward.
- **F3** (fixed differently, in one part): cards 14 and 20 define the seam as uncommitted or untracked paths, and card 20 adds an untracked-only tree to its spawns-nothing list.
  Cards 15 and 21 build the clean-tree read from `fabricengine.Open(...)`'s `Status`, filtered through `loomengine.PathsOutsideRunRecords`.
  The fix key said to join each entry to its side's checkout root.
  I join both sides' entries to `location.WorktreePath()` instead.
  `PathsOutsideRunRecords` drops paths under `shedrun.RunDir`, `LoomDurableDir` and `LoomReviewsDir`, which resolve under `l.AnchorPath()/_lyx` (`internal/shedrun/paths.go:73`, `internal/loomengine/config.go:284`), the warp worktree's junction.
  A weft entry joined to the weft root (`fabricengine.RecordsWorktree`) would never fall under them, so the run's own records would always read dirty and every hand-over would hand back.
  Card 15 states that reason.
  Card 7's tamper snapshot keeps the tracked `DirtyPaths` read.
- **F4** (fixed as proposed): card 12's recheck says that a status decided through `stopFromStatus` is saved through `SaveEnvelope` as a synthesized envelope (`state`, empty `reason`, `loop` with `current_producer` and `history_length`), and that its path becomes the stop's `EnvelopePath`.
  Card 13 adds the moved-to-awaiting `Arbiter` row under Moves and tampering.
- **F5** (fixed as proposed): card 15's Bound now names the real bounds: the steps go only through `RunUntilStop` and `RunGoto`, there is one verify goto per run, the spawns are capped, the successor runs once per stop key and build over a clean tree, nothing moves under a pause, and the driver lock allows one driver.
- **F6** (fixed as proposed): card 3 no longer creates `IsTacklerStrand`, and its tackler.go bullet and `TestTacklerRecords` rows drop it.
  No other plan card names it.

## Disputed

None.

## Deferred

None.
