# Review the change description, bar tooling names, sync the PR body

The change description becomes the PR title, the PR body and the landing commit message.
Until now only its format was checked, an edit on GitHub could leave the PR out of step with it,
and a timed-out `Describe` session halted the run as `failed`.

## What changes

**Tooling names are refused.**
The new `landing.describe_forbidden_terms` key holds regular expressions for names a repository's reader never sees.
The `description` gate, `lym loom validate-description` and `lym beam describe` refuse each matching line,
naming its number and the matched text.
The appended credit block and regression check are not checked.
The `Describe`, beam-describe and darn writer prompts state the rule, as do the briefs of sessions that write repository files.
The Code-Review rubric flags such names in documents and in the description.

**`Describe` gets a reviewer.**
An assistant seat on `landing.describe_review` checks the text for tooling names, unsupported claims,
missing deliverables and a wrong `## Merge danger` call.
It starts reading once the description passes its mechanical check.
The writer's turn end waits for the report, and findings go back to the writer once.
An empty `describe_review` runs no reviewer.

**The PR follows the description.**
PR-Review compares the title and body GitHub holds with what the description yields, at every poll of an open PR,
and sets them again when they differ.
`Finalize` does the same before it merges a recorded PR that is not a beam's.
A hand edit of the title or body on GitHub is therefore overwritten.
While the text cannot be built, or the edit fails transiently,
neither an approval nor `lym loom pr-continue` ends the review.

**Timeouts and interrupted turns.**
A seat step whose chair started and timed out is now `Stuck` instead of an engine error.
Its reason names the elapsed time, the limit, the last session state and `lym loom resume` as the way forward.
A timed-out writer seat's reason carries the same details.
When a turn the operator started ends with output files missing, an autonomous run types one reminder per run,
naming the files.

Closes #227
Closes #228
Closes #241
Closes #244
Closes #255

Part of #225: this covers its first two items, the review of the description and the PR body sync.
Its other four items are not in this change.

## Design decisions

- The term check is a configurable list, because a repository may own some of these names.
  The check at `Publish` and `Finalize` is unchanged: neither runs it.
- The review never fails the step.
  A reviewer that does not finish, or whose time runs out, leaves the description as written, with a log line.
- `describe_review_timeout_min` bounds an unbroken wait; the wait starts over after a gap between checks.
- While a reviewer runs, the writer's and the reviewer's session limits are each
  `describe_timeout_min` plus `describe_review_timeout_min`.
- The chair-timeout and reminder changes are not specific to `Describe`:
  the first applies to every seat step, the second to every autonomous run.

## To check by hand

- The reviewer's time is still counted from the start of the row.
  Both session limits start when the seats spawn (`describeReviewSeat` in `internal/landingshed/describereview.go`,
  `DescribeTable` in `internal/landingshed/describe.go`).
  A writer that runs past `describe_timeout_min` eats into the review's time, down to none at the combined limit.
- LoomYard's own hub owns the names in the default patterns, so set `landing.describe_forbidden_terms` to `[]` there.
- The diff adds no test that drives a live reviewer session, the typed reminder in a real pane,
  or the title and body comparison against real GitHub.

## Merge danger

**Door:** two-way
**Blast radius:** landing
