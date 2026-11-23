# Halt the PR-Review wait when its session ends while the review is open

The PR-Review row's wait judged only the pull request's status, the pull request record and the task worktree.
It never checked whether the session it reviews in was still running.
When that session's strand was removed, the row went on waiting with status running and no agent to commit fixes or end the review.
Nothing ended it until someone killed the `lym loom step` process by hand.

Now each wake that leaves the review open reads whether the session the call keeps is live.
When it is not, the row halts with a reason that names the way forward: `lym loom resume`, which re-enters the row and resumes the recorded session by its session id.
A halt reaches the parent as any judged stop does, so an autonomous run's parent can act on it.

## Design decisions

- **Halt, not re-spawn.**
  The board entry allowed either.
  A strand removed by `lym reed remove` or ended by the operator is often deliberate, and re-spawning it at every wake could loop on a session that dies at start.
  A halt hands the call to the parent, and the resume path already resumes the recorded session.
  The Sampler-Session row handles a session that ends mid-build the same way.
- **Only a session the call keeps is watched.**
  `wait` takes a `keepsSession` flag.
  It is set when the call started or reused the session, and unset for a wait entered on a review already ended or a pull request merged at entry.
  In those cases the record can still name an old dead strand, and halting on it would be wrong.
  A record whose review has ended is never watched either.
- **Where the check runs.**
  It runs on a wake that read no status and on a wake whose judgement left the review open.
  A wake that stopped and resumed the session around its read skips the check, since the resume just made the session live.
  A merge, approval or continue flag that the status read sees still ends the review before the check runs.
  So an agent that runs `lym loom pr-continue` and then exits still ends the review cleanly.

## Tests

`TestPRReview_Wait` gains two rows in `internal/landingshed/prreview_wait_test.go`:

- `ASessionThatEndsWhileTheReviewIsOpenHaltsNamingTheResume`: the session dies after the first wake.
  The next wake halts naming `lym loom resume`, with one start and no stop.
  On the merge base this row waits until the test cancels it, and fails.
- `AReviewEndedAtEntryKeepsNoSessionSoItsDeadStrandDoesNotHaltTheWait`: a review ended at entry, whose record names a dead strand, keeps waiting through a transient poll failure.

## Merge danger

**Door:** two-way
**Blast radius:** landing
