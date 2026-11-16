```yaml
lens: code-correctness
outcome: reported
```

My task was the code-correctness lens: run each branch the diff adds through concrete inputs. I found no defects.

**Branches and the cases I ran**

- `refuseProject`, entry is not a project (`project=false`): returns `(false, false, nil)` before the new block. Unchanged and correct.
- Grouping project, `found=false`, `filed=true` (a mark with no run on record): the new block is skipped and the grouping-project refusal is returned, `judged=true`. Correct: a fresh start of a grouping project is still refused even when a stray mark sits on disk.
- Grouping project, `found=true`, `FilingMarked` nil: the new block is skipped and the run is refused. This matches the field's doc ("a nil FilingMarked reads every run as one that filed nothing").
- Grouping project, `found=true`, `FilingMarked` returns an error (say ENOTDIR from `os.Stat`, since `shedrun/filingmark.go:41-47` returns every error except not-exist): returns `(false, true, probeRefusal(...))`. `refuse` returns that error at `startguard.go:323-324` instead of falling through to the grouping-project refusal. The text matches its neighbours: "could not read the run's filing mark of &lt;slug&gt;: &lt;err&gt;; way forward: fix the cause, then run "lym batten &lt;verb&gt; &lt;slug&gt;" again". Correct.
- Grouping project, `found=true`, `filed=false`: refused naming `lym beam create`. Correct.
- Grouping project, `found=true`, `filed=true`, producer `Run-Shed`, source dirty, every pair part present: returns `(false, false, nil)`, so `refuse` goes on to the task checks:
  - `Unreleased` and `Raddle` run only when `!found`, so they are skipped.
  - The dirty-source check needs `!found`, or a run at Worktree-Create with no task worktree. This run is at Run-Shed, so it is skipped.
  - `PairParts` runs only when `!found`, so it is skipped.

  The guard passes. Correct: a filing happens inside Run-Shed, so a filed run is always past Worktree-Create.
- Same run with the status at done, as after the inner loom run finishes and batten's run completes:
  - `refuseAtStart` asks `clearsMotifRunOfRaddleEntry`. For a project entry, `startClearsMotifRun` reads the board recipe, then the child recipe. The child recipe is `arras`, not `motif`, so `found` stays true.
  - The guard passes through the filed branch.
  - The shed loop stops at once on a done state (`shedverbs/loop.go:316`).

  Harmless.
- The `Promoted` check, which runs before `refuseProject`, reads false for a project entry (`startguard.go:108`), so it cannot refuse the filed run.
- `hubStartGuard` wiring: `armAt` sets `location = prime` before calling `hubStartGuard(location, runID)` (`arm.go:332-341`). The mark is written with `shedrun.WriteFilingMark(prime, slug)` (`loomcli/raddlewiring.go:264-270`). The guard reads `shedrun.FilingMarked(location, slug)` from the same prime and slug, and `RunDir`/`runSegment` resolve them the same way the status file does. The read is for this slug only, never for a parent or sibling. Correct.
- Stale mark: both `ClearMotifRun`/`clearDoneRun` and a cancel with `--remove` delete the run directory, and the mark with it. A leftover mark with no status file only reaches the `found=false` branch, which still refuses. Correct.

**Findings:** none.

**Tool calls**
1. Searched `internal/shedverbs` for `StateDone`, to see what a `run` verb does with a done status the guard now lets through. My context did not hold the shed loop.
