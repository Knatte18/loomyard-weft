# Webster-Burler round 1 — fixer report

Every finding in `round-1-review.md` is fixed, one commit per finding.
After the last fix the plan's full `verify:` block passes on `1225c84b4`: build, the untagged and `integration`-tagged tests, and `go vet -tags smoke`.
Nothing was pushed.

## Fixed

- **F1 (MEDIUM), `049c7a931`.**
  `teardownRefusal`'s dirty-sibling remedy still names `lyx loom commit-records` then a resume.
  It now also says what to do when the resumed teardown refuses again: the leftover is not a run record, so inspect the pair from the task anchor and commit or remove it by hand, never `--force`.
  The doc comment explains why: fabric's refusal covers any uncommitted content in the sibling, not only what `commit-records` stages.
  docs/overview.md no longer claims that "other uncommitted content keeps fabric's refusal".
  It now says that uncommitted content in the task worktree itself keeps fabric's refusal.
  `wire_test.go` asserts the new wording.
- **F2 (LOW), `23da48ba1`.**
  The failed-archive remedy now reads "fix the failure named here (most often an unreachable remote)" instead of "make the remote reachable".
  The wrapped cause is still carried with `%w`, and `wire_test.go` asserts the new wording.
- **F3 (LOW), `e226003d7`.**
  `callDone` now treats a done-seen marker that reads but does not parse as a fresh first sight.
  It logs a warning naming the path and the parse error, rewrites the marker with the current time, and continues the wait.
  A read failure other than "absent" is still a hard error.
  New test: `TestInnerRun_UnparseableDoneSeenMarkerRestartsTheGrace`.
- **F4 (LOW), `ecf54ecca`.**
  docs/overview.md's batten `status` sentence now says the self-bounce appends one entry per poll while the child runs, awaits approval or waits for its driver, so the history outgrows the 1440-bounce running budget.
- **F5 (LOW), `1a053cdf1`.**
  `records_integration_test.go` now asserts the teardown step is `Worktree-Teardown` with outcome `done` and run state `done`.
- **F6 (LOW), `1225c84b4`.**
  The new and changed Go comments in 42 files are rewrapped to one sentence per line.
  Breaks the authors had already placed after a semicolon or colon, and at a deliberate clause break on an unwrapped line, are kept.
  A sentence was rewrapped only where it touches a line this branch added or changed.
  Untouched legacy sentences are left alone, except where a legacy sentence shared a line with a changed one.
  The commit changes only comment line breaks: the removed and added text match word for word (4467 words each), gofmt is clean, and no non-comment line changed.
  A re-scan of the branch diff finds no remaining mid-sentence comment break.
- **F7 (NIT), `0b6b7ba16`.**
  Dropped "Row names and max_bounces are unchanged." from `contracts/recipes/batten-recipe.yaml`'s header comment.

## Deferred

None.
