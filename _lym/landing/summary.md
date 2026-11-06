# Drift: stop lym's own writes and leftovers halting runs

Four causes halted drifting runs in the Models hub at Drift-Verify and Drift-Recheck.
Each is fixed in `internal/driftengine`, with the preflight wiring in `internal/preflightshed` and `internal/loomcli`.

## What changes

- **The judge guard no longer halts on lym's own records.**
  `resumeJudgeSession` cleared `scope.SnapshotLost` after taking the snapshot, so the snapshot-versus-checkout comparison named that write as the judge's.
  It now clears the mark first and then takes the snapshot.
  `openJudgeSession` likewise takes the snapshot before it opens the session mark, so an open mark always has a snapshot.
  `JudgeSnapshot.Changes` also exempts the drift logs, and `scope.json` and `judged.json` while both match their newest Drift-Records commit.
  A hand edit of either record, a task file or any other record still halts.
- **A gone snapshot no longer halts a resume that guards nothing.**
  An open session whose snapshot is gone and whose judge chair is not live is marked lost and gets a fresh snapshot, as the halt text already promised.
  Under a live chair the halt stays, and its text says what cannot be told.
  `TakeJudgeSnapshot` clears only the untracked copies and replaces the record atomically, so a reader never finds the snapshot missing while one is taken.
  The guard halt's way forward no longer tells anyone to delete the snapshot directory, which is what lost it; it says to keep that directory, and names only the current capture's scratch directory for deletion.
- **Volatile capture provenance is ignored.**
  `drift.yaml` gains the hub-wide lists `volatile_files` (globs, matched against the file name at any depth, or the whole path when the glob holds a `/`) and `volatile_keys` (line keys as the `key_patterns` read them).
  Under `diff: lines` the diff drops those cells and listings before it classifies them.
  Both are guard keys, refused to a task's run settings, and documented in the template.
  A run records the lists in its scope record and keeps them.
  A blank entry or an invalid glob is refused at config load.
- **A retired `{skip_file}` is migrated out and found early.**
  `lym config reconcile --layer default --apply` removes the whole `-e NAME={skip_file}` argument (also `--env NAME=…` and `--env=NAME=…`, quoted or not) from the `capture` and `compare` commands.
  Any other use of the placeholder stays for the operator, as does an argument that ends a quoted shell string such as `sh -c '… -e A={skip_file}'`.
  A new `driftengine.CheckRetiredPlaceholders` runs as a Preflight check for recipes with Drift rows, so a leftover stops the run before its first agent.
  The refusal now names a way forward that works from a run session: the operator or hub orch runs the reconcile from the prime worktree, then `lym loom resume`.

## What the Models hub must set

Add the provenance file to the hub's `drift.yaml`:

```yaml
volatile_files: [_source.txt]
volatile_keys: []
```

Use `volatile_keys` (for example `[commit, captured_at]`) instead if the keys, not the whole file, differ in every capture and other keys in `_source.txt` should still count.
Then run `lym config reconcile --layer default --apply` once to drop the `NORCE_BASELINE_SKIP_FILE={skip_file}` arguments from `capture.command` and `compare.command`.

## Design decisions

- lym hard-codes no provenance file name; the file belongs to the hub's harness, so the lists live in the hub's config.
- The lists are guard keys: they decide what counts as drift, so a task may not widen them.
- The `{skip_file}` migration removes only the `-e NAME={skip_file}` argument form, since lym cannot tell what any other use of the placeholder means.
- Exempting `scope.json` and `judged.json` is conditional on matching their newest Drift-Records commit, because every lym write lands in such a commit.

Closes #69
Closes #70
Closes #71
Closes #74
Closes #75
Closes #76
Closes #77
Closes #78
Closes #79

## To check by hand

Nothing here ran against a real Models hub.

- In the Models hub, run a drifting task through a lost-snapshot resume and a new capture of a new commit, and confirm neither halts.
- Run `lym config reconcile --layer default --apply` against the hub's `drift.yaml` and confirm both `-e NORCE_BASELINE_SKIP_FILE={skip_file}` arguments are gone and the rest of each command is intact.
