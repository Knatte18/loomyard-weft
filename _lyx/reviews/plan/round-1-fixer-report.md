# Fixer report — plan, round 1

## Changes

- **F1** (fixed as proposed): card 2's Intent now has the implementer update `internal/battencli/cli.go`'s whole `lyx batten overview` package-doc paragraph.
  That means putting the orch line first, with its fields, its absent case and its two-strike `orch  state unreadable`, and naming the orch state file and orch config in the "only reads" sentence.
- **F2** (fixed as proposed, with one small extra): a stat that fails with `os.ErrNotExist` now leads to `PeekState`, and `PeekState`'s not-found result is the only absent branch.
  Frames 1 and 12 of `TestOverviewSource_DrawsTheOrchLine` reach that branch.
  The old untested race-guard sentence and the absent-file clause in the stat sentence are gone.
  Beyond the proposed text, the failure sentence now reads "Any other failed stat", so it no longer covers the not-exist case.
- **F3** (fixed as proposed): step 11 writes the good state at a new modification time and expects `orch  212k / soft 300k / hard 400k  idle  cycles 3`, then `runs unreadable`.
  Step 9 is now an unnumbered sentence after step 8 saying that no frame so far created `StateLockPath`.
  Steps 10–12 keep their numbers, so the cross-references still hold.
  The `WatcherExit` row of `TestRenderOverview` now sets 212500 tokens, idle and 3 cycles, and pins the whole line `orch  212k / soft 300k / hard 400k  idle  cycles 3  watcher down`.

## Disputed

None.

## Deferred

None.
