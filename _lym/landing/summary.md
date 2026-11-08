# PR-Review asks the parent orch for a review behind pr_review_ask_parent

The operator finds the parent orch's review of a run's pull request useful, but gets one only by telling the PR-Review agent to ask, or by asking the orch directly.
A new `loom.yaml` key, `pr_review_ask_parent` (default `false`), makes the PR-Review row ask on its own.
It is open to the local layer, so a task's `run_settings` turns it on or off for one run.
With the key off, PR-Review runs as before.

Closes #86

## What changes

- `internal/loomengine`: `Config.PRReviewAskParent`, the `pr_review_ask_parent` key in `ConfigSpec` and its line in `template.yaml`.
- `internal/landingshed`: once the row has the open pull request and its session, and before it waits, it mails the parent orch a line opening `[pr-review request]`.
  The line names the slug, the pull request URL, the task and parent branches, and the PR-Review session's strand, which the orch answers.
  Two new `PullRequestDeps` seams carry this: `ParentReviewWanted` (nil reads as off) and `AskParent`.
- `internal/pullrequest`: `Record.ParentReviewAsked` keeps the ask to once per record.
  A review started afresh writes a new record and so asks again.
- `internal/loomcli`: `ParentReviewWanted` reads the key over the pair's layers, so `run_settings` applies.
  `AskParent` goes through `orchcli.NotifyParent`.
- `contracts/orchrole/orch-role.md`: a "Reviewing a pull request" section tells the orch how to answer a request.
  It judges the change against what the run needs, answers the named session with satisfied or findings, and never approves, merges, closes or comments.
- `contracts/stencils/landing/landing-template-pr-review.md`: the PR-Review agent is told that its parent may review.
  It fixes the orch's findings like the operator's, commits, and asks the orch to look again until it is satisfied.
- `docs/overview.md` and the `landingshed` package doc describe the setting and the row's behavior.
- `internal/testkit/envkit` fills the new seams.
  The large diffs in `internal/battencli/wire_seams_test.go` and `internal/loomcli/wiring_seams_test.go` are gofmt realignment from the longer map key.

## Design decisions

- The ask is non-fatal.
  An unreadable setting, a failed post and an orch no record names are logged, and the next entry of the row asks again.
  A failed write of `ParentReviewAsked` after a successful post is logged and not returned, so the driver does not re-step and post a second request.
- The row asks and does not gate.
  The orch's answer reaches the session's agent as creel mail that the row never reads.

## To check by hand

- The board entry says that with the setting on, the run waits at PR-Review until the orch answers satisfied or sends findings.
  The row does not do this: a merge, an approval or the operator's `lym loom pr-continue` still ends it whether or not the orch has answered.
  Decide whether the operator's ending overriding the orch is acceptable, or whether the row must hold until the orch answers.
  Holding would need the answer recorded in `pullrequest.Record` and `judge` in `internal/landingshed/prreview.go` to wait on it.
- Run the flow once against a real run with the key on, to see the orch receive the request and the agent act on its reply.
  No test covers the live exchange.
