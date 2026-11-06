# Drift: four false halts of Models hub runs are gone

Four bugs halted drifting runs in the Models hub at Drift-Verify and Drift-Recheck.
Each was checked against `internal/driftengine` as it stands after `98bf70b0`, and each still held.

## The judge guard read lym's own writes as the judge's (issues 74, 75, 77)

`resumeJudgeSession` took the new snapshot and then cleared `snapshot_lost`, so the guard named that write as the judge's and the designed recovery always halted.
It now clears the mark first and then takes the snapshot.

The order alone would leave the same halt behind other writes: a `lym drift amend` or a recorded check while a session mark is open, and each compare log a check under `diff: command` adds.
`JudgeSnapshot.Changes` therefore exempts what lym writes:

- `_lym/drift/logs/`, and
- `scope.json` and `judged.json`, while both match their newest `Drift-Records` commit.

Every lym write of those two records lands in such a commit, so the digest tells lym's write from an edit.
A hand edit, or a commit without the trailer, still halts naming the file, and the next entry then halts on the records mismatch with `lym drift amend <slug> --restore`.

## A gone snapshot halted the run (issues 76, 78)

- The neighbouring halt told the reader to delete `the changed scratch directory`, which was read as the snapshot directory.
  The text now names the scratch directory by its path and says to keep the snapshot directory, which the resume compares against.
- An entry that finds the snapshot gone probes the judge's chair.
  With no live chair it records the snapshot lost and goes on, so the resume takes a new one under the recorded spawn id.
  Under a live chair the halt stays and says the session's changes cannot be told, and so does a snapshot that went while the judge ran.
- `TakeJudgeSnapshot` no longer removes the whole directory first.
  It clears only the untracked copies, and one atomic write replaces the record, so a reader never finds the snapshot gone while one is taken.
- `openJudgeSession` takes the snapshot before it opens the mark, so an open mark always has a snapshot.

## Capture provenance read as drift (issues 71, 79)

`drift.yaml` gains two hub-wide guard keys, documented in the template, and lym hard-codes no name:

- `volatile_files`: globs of capture files the diff leaves out whole, matched against a file's name at any depth, or against its whole path when the entry holds a `/`.
- `volatile_keys`: line keys the diff leaves out of every file.

`LoadConfig` refuses a blank entry and an invalid glob, and a task's `run_settings` are refused both keys.
A run records the lists with its diff mode, so a later config change never moves it.
Check drops their cells after the diff, and the expect agent's listing leaves them out so it writes no expectation of them.
The baseline digest still covers these files, and under `diff: command` the declared compare decides alone.

**What the Models hub must set**, from the prime worktree:

```
lym config drift --set 'volatile_files=[_source.txt]'
```

That covers `commit`, `captured_at` and everything else in the file.
A run whose scope record already holds a diff mode keeps its empty lists, so an in-flight run still needs its `lym drift amend`.

## A retired `{skip_file}` stayed in hub config (issues 69, 70, 79)

- `MigrateConfig` now removes each whole argument that hands the placeholder to a variable, in the forms `-e NAME={skip_file}`, `--env NAME={skip_file}` and `--env=NAME={skip_file}`, quoted or not, from `capture.command` and `compare.command`.
  It runs at `lym config reconcile --layer default --apply` and in the hub walk of a new build, so the Models hub's file clears itself.
  This is a migration and not a reinterpretation: the placeholder's meaning already changed, and a reinterpretation's marker would also keep it from running at an operator's reconcile.
- Any other use of the placeholder stays, since lym cannot tell what the command means by it.
  `CheckRetiredPlaceholders` finds such a leftover at the Preflight row of a recipe with Drift rows, through a new `Drift` check, before any agent runs.
  It checks only while drift is enabled, and checks the compare only under `diff: command`, the one mode that runs it.
- `LoadConfig` does not refuse the placeholder.
  A refusal there would fail the load check of every `lym config drift --set`, so a hub with both steps holding it could not fix one step at a time.
- The refusal's way forward now hands the reconcile and the step's edit to the operator or the hub orch, since a beam orch or task session is refused hub-wide config writes.

## Worth a reviewer's eye

- The exemption of the drift logs is a bound: a judge's edit under `_lym/drift/logs/` goes unseen. They hold no code, no expectation and neither record; the package doc states it.
- The two drift `{skip_file}` constants became one, and `encodeDocument` is shared by the two `drift.yaml` rewrites.
- The stencils and drift Preflight checks share `prefixedFailure`, so the stencils failure's log line now reads `a check failed` with a `check` field.
