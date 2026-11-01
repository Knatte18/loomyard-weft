# batten cancel hangs on a step held by its own pause, and --remove fails on removed records

`cancel-hangs-on-own-pause` · Task · bug, batten

## Seen (orch, 2026-11-01)

The operator pressed cancel on `plan-spec-split` while batten was at Open-IDE, a few seconds after Seed-IDE.
The button's `lym batten cancel plan-spec-split` paused the run and then hung: the running `lym batten step plan-spec-split` (Open-IDE) waited on that same pause ("held by the pause of plan-spec-split ... continues after lym tender resume"), so the cancel waited on the step and the step on the cancel's pause.
Only killing the step let the cancel finish (`state failed at Open-IDE`, pair kept).
Then `lym batten cancel plan-spec-split --remove` removed the worktree pair and branches but failed at the end: `git add -- _lym/shed/plan-spec-split: pathspec did not match any files`, since the records directory was already gone.
So the remove is not idempotent and reports failure on a clean result.

## Idea

- A cancel never waits on a step held by its own pause: it ends the step or does not pause before it.
- `cancel --remove` treats an already-removed records directory as done and reports success.

## Cause of the remove failure (orch, 2026-11-01)

`RemoveRecords` in `battencli/cancelcmd.go` runs `os.MkdirAll` for the status lock's directory, which recreates an empty `_lym/shed/<slug>` after the records are gone, and `removeRunRecords` then runs `git add -- _lym/shed/<slug>`, which fails on the empty directory.
Every rerun fails the same way, so the cancel marker (`<weft>/.lym/shed/<slug>.cancel.json`) stays, and dobby refuses to start the task again ("a cancel of <slug> was interrupted").
The orch cleared it by deleting the marker and its two lock files by hand, after checking that the marker's process was dead and the claim, pair and records were gone.
The fix: the records removal treats a missing or empty records directory as done.
