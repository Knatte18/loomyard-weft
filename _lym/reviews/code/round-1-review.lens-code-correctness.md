```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens. I traced each function and branch the diff adds or changes, using concrete inputs including boundary and failure cases.

**Cases run**

- `readBoardHistory`, zero floor: one `Log` over the whole history and no prior read. Correct.
- `readBoardHistory`, non-zero floor:
  - No commit at or after the floor: `before = tip`, so the board in effect at the floor is the tip. Correct.
  - Commits after the floor: `before` is the first parent of the oldest commit walked. Correct.
  - The oldest commit walked is a root commit: `before = ""`, so there is no prior read. Correct.
  - A failing prior `Log`: the error is wrapped with the floor time. Correct.
- `readBoardHistory`, how much it walks: a non-first-parent `Log` with `Since` still visits every commit of the branch, so it is not bounded by the window. Finding 1.
- `boardTasksAt`:
  - An absent path returns nil, nil.
  - A board that does not parse returns nil, nil, as `tasksAt` did before.
  - Any other read error is wrapped.
  - Correct.
- `boardHistory.add`:
  - Two entries of one slug, the second reading done: done is marked and labels come from the first entry, as the old code did.
  - Labels that change at one commit open a new span.
  - An entry missing at one commit ends its span.
  - Correct.
- `extendSpans`:
  - Empty spans get a new span.
  - The next index extends the newest span.
  - A gap opens a new span.
  - Correct.
- `doneAfter`:
  - `open` lies before the floor: the prior commit is older than every `open`, so it never counts.
  - A slug reopened after done: the first done commit strictly after `open` is returned, in the old order.
  - Correct.
- `labelsAt`:
  - `at` lies between spans: the newest commit at or before `at` is found. Correct.
  - The entry is held only before the floor's board: no labels and a finding, where the old code found labels. This is declared. Finding 3.
- `readLandings`, floor:
  - Zero `Since` keeps a zero floor whatever the owners. Correct.
  - An owner opening before `since` lowers the floor to it. Correct.
- `timeJoinOwners` and `joinTime` split: same owners and same claims. Correct.
- `assemble` no longer returns an error: no error path is lost, because the reads moved into `readBoardHistory`. Correct.
- `newBattenIndex`: a zero `windowStart` gives a zero floor and a walk of the whole history. Correct.
- `walk`: it rebuilds `touching` and `checkpoints` on every walk. Correct. The checkpoints it collects are bounded by the floor. Finding 2.
- `widen`:
  - A zero floor returns false.
  - A doubled walk with an unchanged count falls back to a walk of the whole history, after which the floor is zero and the loop ends.
  - The loop terminates.
  - Correct.
- `battenStatusReader.at`:
  - The cache is per sha.
  - Read errors are not cached and propagate.
  - An absent path or a status that does not decode is recorded as not held.
  - Correct.
- `isFreshSeed`:
  - k > 0 reads `commits[k-1]`.
  - k == 0 with a zero floor or a parentless commit counts as a seed.
  - k == 0 with a non-zero floor reads the status at `Parents[0]`, which equals the slug's status before that commit.
  - Correct.
- `newestSeed`: commits after `until` and at or before `earlier` are skipped, and -1 comes back when there is none. Correct.
- `snapshotsOfRun`, reused slug (test row 1): the read at `gap` decides that `b300` is fresh. Correct.
- `snapshotsOfRun`, seed before the floor with `earlier == nil`: it widens until the seed is found or the whole history is walked. Correct.
- `snapshotsOfRun`, no seed with `earlier` at or after the floor: it breaks with no widening, which is correct because only commits after `earlier` qualify.
- `snapshotsOfRun`, no seed at all: nil snapshots, as before. Correct.
- `ComputeTiming` and `selectInFlight`: they share one index, and a widen during `selectRuns` only adds commits. The runs and spans either set sees are the same as before. Correct.

**Findings**

1. **MEDIUM · design** · `internal/reportengine/boardhistory.go:43`, and the same at `internal/reportengine/batten.go:74`
   - A walk with `Since` but not `FirstParent` still traverses the branch's whole history, so the board read is not bounded by the window.
   - Evidence: `internal/gitrepo/historyreads.go`, non-first-parent `Log`. `commitIter.ForEach` visits every commit and only skips those before `Since` (`return nil`); it never stops.
   - Effect: the board walk still loads all of the roughly 26 000 board commit objects. The records walk does the same, and each `widen` re-walks the whole records history.
   - The parsed boards are no longer held, so the memory blow-up is fixed. The amount of history walked is still proportional to the whole history, which contradicts the Fix's "bounded by the report window". The fake `Log` (`fake_test.go:209-215`) mirrors this skip, so no test can see it.

2. **MEDIUM · design** · `internal/reportengine/batten.go:91-94`, feeding `runs.go:210`
   - Landing checkpoints now come only from the bounded records walk, so a checkpoint older than the walk's floor opens no time-join window.
   - Whether it is seen also depends on whether an unrelated run's seed widened the walk. `widen` rewrites `b.checkpoints`, and `selectRuns` captures the checkpoints only after every `snapshotsOfRun` call.
   - Most exposed are partial runs and untagged pair slugs, which never call `snapshotsOfRun` themselves. Their landings can turn unmapped, or turn mapped depending on another run's history.
   - Evidence: `runs.go:163-166` skips `snapshotsOfRun` for a partial run, and `landings.go:298-325` reads `inputs.checkpoints`.

3. **LOW · design** · `internal/reportengine/boardhistory.go:449-460`
   - `labelsAt` gives no labels, plus a "no board commit holds an entry" finding, for a slug absent from the floor's board and every board after it but held earlier.
   - The old `labelsOf` (diff piece 001, removed lines 738-754) found such labels.
   - The summary declares this change, but it changes the kind and the findings the report prints.

Tool calls:
- `sed -n 148,240p internal/gitrepo/historyreads.go`: the real `Log` implementation, to see how `Since` bounds a non-first-parent walk. My context held only the `LogQuery` doc comment.
