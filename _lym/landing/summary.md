# Add `lym beam reopen`; name PRs in squash commits, list sub-PRs

Closes #46
Closes #44

## Reopen a beam from PR-Review

A beam's pawl run could not take new work once it left Beam-Await.
`lym beam reopen <slug>` now sends a landing back in one step.

- The verb refuses, each time naming a way forward, an entry that is not an open beam project with a pair here, a beam another machine owns, a beam with no pawl run, and a run at any row but Parent-Align, Beam-Describe or PR-Review.
- It writes a reopen request naming the run into the beam worktree's `.lym/beam` directory, then removes the beam's ready record and commits that, so the dobby does not land the beam again.
  `pawlengine.RemoveReady` is the new removal; an absent record, or one for another slug, is left alone.
- The PR-Review row answers the request on entry and at every wake.
  It stops the pr-reviewer, converts the open pull request to a draft and comments that the beam was reopened, then reroutes the run to Beam-Await.
  `--close` closes the pull request with a comment instead, and drops the record, so the next landing opens a new one.
- The pawl recipe gives PR-Review `on_reroute: Beam-Await`.
- Beam-Await removes a request on entry, starts the beam's dobby when it is down, and posts that the beam is reopened.
- The next PR-Review reuses the kept pull request and marks it ready for review before the review starts, since GitHub refuses to merge a draft.
  Beam-Describe refreshes its body.
- `pullrequest.API` gains `SetDraft` and `Comment`, and `Status` gains `Draft`.
  GitHub's REST API cannot set draft state, so `SetDraft` reads the pull request's node id and runs the `convertPullRequestToDraft` or `markPullRequestReadyForReview` GraphQL mutation.

Design decisions:

- The request is a file the PR-Review row consumes, as the land request is for Beam-Await, so a halted run takes it once `lym loom resume` resumes it.
- PR-Review leaves the request in place and Beam-Await removes it, so a run the reroute cap halts at PR-Review answers it again when resumed.
- If Beam-Await cannot remove a request naming its own run, it starts nothing, posts one notice naming the file and retries at the next wake; otherwise PR-Review would read the stale file as a fresh request and reroute at once.
  A request naming another run is removed on a best-effort basis.
- A pull request GitHub reports as merged or closed loses its record, so the next landing does not judge the reopened work as landed.

## Name the PR in squash commits, list sub-PRs

- Finalize's merge on GitHub ends the commit subject with ` (#<number>)`, the form GitHub's own squash merge writes.
- Beam-Describe now renders a `## Changes` list, oldest first, from the commits on the beam branch's first-parent line that its parent lacks.
  `BeamDescribeDeps.CommitSubjects` becomes `Commits`, returning `BeamCommit` (full SHA, message, committer time).
- A commit whose subject ends in ` (#<number>)` is listed as `#<number> <title>, merged <time>`.
  Any other commit, a merge from the parent included, is listed as `<12-char SHA> <subject>, committed <time>`.
  GitHub links both forms in the body.
- Each entry carries the first sentence of the commit body's first non-heading paragraph as its summary, cut at 200 characters.
  Times are UTC.

The PR numbers come from commit subjects, so a sub-task landed before this change lists by SHA, not by number.

## To check by hand

- Run `lym beam reopen` against a real beam with an open pull request, with and without `--close`.
  The GraphQL draft mutations and the comments are covered by tests of the client and the row, not by a call to GitHub.
- Reopen a run at Parent-Align and Beam-Describe, and a halted run, and confirm each reaches Beam-Await once it gets to PR-Review or is resumed.
- Confirm the rewritten PR body from a real beam shows the `## Changes` shape of the example in #44.
