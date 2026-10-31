# Dobby posts a landing notice per land_notice and notify_on_land

A run that landed sent nothing, so the orch learned of a landing only when something else woke it.
Now the dobby that drives a run posts one creel line to its worktree's nearest live orch once the run lands or ends without landing:

```
[dobby notice] <slug>: landed as <commit>
[dobby notice] <slug>: ended without landing; lym batten status <slug> shows the run
```

## What changes

- **One past-Land test.** `battenrecipe.PastLand` is the rule hibernate's `landFinished` held (done, or at Worktree-Teardown in any state); `landFinished` is gone and hibernate's after trigger calls `PastLand`, unchanged in behavior.
  `battenrecipe.LandOutcomeOf` also tells a run that landed, whose history holds a Land Done, from one that ended without landing.
- **beam.yaml `land_notice`**, beside `max_runs`: `all` sends a line for every run the dobby drives, `per-task` (the default) only for a run whose board entry asked for one, `off` none.
  Any other value is refused naming `lym config beam --set land_notice=<all|per-task|off>`.
- **Board field `notify_on_land`**, set with `lym board upsert`.
  It is not scope, so a run-held entry takes it and the revised date ignores it; a note carrying it is refused.
  `Board.ClearNotifyOnLand` clears it, and writes nothing when the entry is gone or does not carry it.
- **The landing commit.** Finalize hands the parent pair's head to a new `RecordLanding` seam right after a parent-side merge that landed, before the push; loomcli writes it to `shedrun.LandedCommitFile` in the prime's scratch of the run's batten slug.
  A failed head read or write is only logged.
- **The dobby step** (`internal/dobbyengine/landnotice.go`).
  Each tick, the dobby remembers in its state record every listed run ahead of its Land row with whether its entry carries the field, since Land prunes a landed entry.
  A remembered run seen past Land gets its line as `land_notice` decides, and the entry's field is cleared once a line for it went out.
  A failed post is retried on the next tick, three attempts in all.
  A beam's batten run is driven by its parent's dobby, so a beam's landing gets its line the same way.
- **The overview.** The dobby header in the reed window and `lym beam status` reads `alive, max_runs N, land_notice <value>`, and a run or waiting task whose entry asks for a notice ends its line with `notify_on_land`.

## Design decisions

- dobbyengine imports no recipe package: `battencli.DobbyRuns` calls `LandOutcomeOf` and hands the dobby `PastLand` and `Landed` booleans on each `RunView`, with the landing commit beside them.
- A run first seen already past Land is never remembered, so a fresh or restarted dobby posts nothing for old landings under `all`.
- A board that does not read keeps each remembered request as it last read, and a beam.yaml that does not load settles no notice that tick; both are logged once per episode.
- The landing commit is the parent's head right after the merge, so a Finalize re-entered after a later landing on the same parent names that newer head.
- `lym board upsert`'s help text was trimmed to stay under its 2000-byte cap with the new field.

## Tests

- `battenrecipe`: `PastLand` and `LandOutcomeOf` over running, blocked, teardown and done statuses with and without a Land Done.
- `boardengine`: the field's storage, the note refusal, `ClearNotifyOnLand`, and a run-held entry taking the field.
- `landingshed`: the head is recorded after the board mark and before the push, never on a stuck landing, and a failed record keeps Done.
- `dobbyengine`: `land_notice` loading and refusal; the notice under each value, the line's text, the request's clearing after the prune, the retry and drop, and the memory's upkeep; the view's new fields.
- `battencli` and `beamcli`: the overview lines, `DobbyTasks` reading the field, and `DobbyRuns` reading the outcome and the commit over a hub (tier1).
