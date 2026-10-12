# Round 1 fixer report: discussion

## Changes

- F1, fixed as proposed: D7 states the no-mutation rule as stencil instruction only, adds Go's post-run check of the stop key and the worktree's uncommitted paths (cause `tampered`), performs a tackler `resume` through the arbiter's consume path (`shedengine.Relaunch` with the under-lock Admit writing the `applied` mark), routes a refusal to D12, and states the wrong-verdict bound.
  Acceptance criteria gain the changed-stop-key consume case and the `tampered` case.
- F2, fixed as proposed: D4 states the full record match batten's `recordMatchesHalt` applies, sends a failed match to D3's last row, and spends the one retry when either batten's publish-retry marker or a Go settle record shows one, the Go driver writing batten's marker state.
  D3's verify row now says a record that matches the halt (D4), and its last row reads "any other `blocked` stop".
  Acceptance criteria gain the stale-record and cross-actor retry cases.
- F3, fixed as proposed: D5 defines the clean-tree check (no uncommitted or untracked path outside the run's records directories, through the existing dirty-paths read; a dirty tree hands back naming the paths), orders the hand-over (settle record, stop report and park marker, then release, then spawn), states the recovery for a crash or failed spawn after release, and records batten's once-per-episode-and-build cap in a marker beside its publish-retry marker.
  The unwatched parent line is sent before the release, since after it no actor remains to send it.
  Acceptance criteria gain the failed-spawn case and the dirty paths in the stop report.
- F4, fixed as proposed: D1 makes `refuseAdoptedSeed` refuse a typed `--child-driver` against a found seed that records none, with its way forward and refusal-spec row; the Seeding criterion and the Constraints refusal list name it.
- F5, fixed as proposed: D13 is rewritten around the three designed empty-reason persists (goto, bare `pause_requested`, cancellation), shed records a reason at each, and the friction note fires only for a reason still empty; the Scope line and the acceptance criterion follow.
- F6, fixed as proposed, first option: D3's transient row goes to the tackler, budget permitting, with a sentence that the loop has already re-stepped it once; the last row drops the transient clause.
  The Notes line now names D3's departures from the LLM driver.
  Because a transient row that always matches would shadow the arbiter row below it, the row for the same stop key after an applied tackler `resume` moves above the transient and `interrupted` rows, so a returning stop key reaches the arbiter as D6 and D7 say.
- F7, fixed as proposed, second option: D4 keeps the pid-only watched read, states its consequence, and has the Go driver send the unwatched hand-back line at every verify halt it hands back as watched; the verify-halt acceptance criterion says so.
- F8, fixed as proposed: deleted the D3 sentence on the `interrupted` under `handback` narrowing and the D6 sentence comparing with `arbiter-when`; the support log's goal-scope and board-fit lens entries now carry the pointer and the `arbiter-when` comparison.
- F9, fixed as proposed: D10 names resume's whole `Long` help, its opening no-spawn statement and the `lyx loom start` refusal branch.

## Disputed

None.

## Deferred

None.
