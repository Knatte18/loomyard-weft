## Goal

```markdown
## What was seen (Neve, 2026-11-22)

`lym report convergence --since 2026-10-20 --format markdown`, run from the prime, grew by about 2 GB every 15 seconds to 9.5 GB resident within two minutes.
Beside the hub's runs it filled the 16 GB user slice and its swap; the load reached 112 and the machine froze at 11:27 and needed a hard restart.
The `memsample` lines in the journal of boot `518480c4e3544695b00b6cff201b8af5` show the growth sample by sample.

## Cause

`readLandings` (`internal/reportengine/landings.go`) walks the whole board branch with no window, `src.Board.Log(src.BoardTip, gitrepo.LogQuery{})`: about 26 000 commits, since every board write is a commit.
`boardDoneAfter` and `labelsOf` then read `board.json` at board commits one by one, a git read each, through `tasksAt`.
`tasksAt` keeps every parsed board in `boardTasks` for the whole report and never drops one, so memory grows with the board's history.
Each `boardDoneAfter` call walks every board commit after its run opened, once per run, so the work grows with runs times board commits.

## Fix

Read the board history once, bounded by the report window, and keep only what the joins need:
per slug, the time its entry first reads done after a given point, and its labels at a landing.
No parsed board is held past the read of its commit.
A test pins the bound: the joins read each board commit at most once, and hold no board beyond the one being read.

## Sweep of the rest of reportengine (operator, 2026-11-22)

Go through every report in `internal/reportengine`, timing, cost, raddle and replies included, for the same pattern:
a git history walk with no window, a read per commit where one read would do, or a cache that keeps every parsed revision for the whole report.
Fix each place found in this run, and pin each fix with a test as above.
```

## Scope

## Decisions

### Added after Discussion (parent, 2026-11-22): F5: bounded reads keep the old join results

Decision: Restore the old landing join results within the bound, as the review's restoring path names: collect landing checkpoints from a walk that does not depend on run seeds, so widening for another run no longer changes them, and in readBoardHistory keep reading older board commits for each claimed slug the floor's board lacks until its entry is found or the history ends, keeping only its labels. The bound limits memory; it never changes what the report answers.

Rationale: The goal is to stop the report freezing the machine, not to change its figures; a report whose results depend on which other runs widened the walk is wrong.

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria
