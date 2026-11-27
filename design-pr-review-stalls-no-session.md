# PR-Review in review mode stalls with no agent session

`pr-review-stalls-no-session` · Task · bug, landing, loom, shuttle · starting · Open-IDE

Imported from [issue #282](https://github.com/Knatte18/loomyard/issues/282).

## Seen

A darn run under a beam (`beam.pull_request: review`, autonomy on) opened its pull request into the beam branch and reached PR-Review.
The row has waited for over half an hour as a bare `lym loom step` process, with no session in the task's tmux window.
The operator's terminal window shows only the loom-status pane, so nothing on screen says the run waits on the operator, and no agent is there to work the pull request with.
Since the pr-reviewer role was retired, `internal/loomshed/interruptpolicy.go` states "PR-Review spawns no session".
The orch cannot end the wait either: `lym loom pr-continue` refuses every lym session.

## Expected

PR-Review should behave in one of three clear modes:

1. **Pull request reviewed by the operator** (`review`): an agent session is in the task's window while the row waits.
   The operator works the pull request with that agent, which answers comments, fixes findings and ends the review on the operator's word.
2. **Pull request opened but accepted automatically**: the row ends the review on its own and goes on to Finalize with no wait.
3. **No pull request**: the row is skipped and the run goes straight to landing.

Today's `review` mode is none of these: it opens the pull request and stalls with no agent and no visible sign of the wait.

## Workaround

The operator ends the review by approving or merging on GitHub, or by running `lym loom pr-continue` from a shell outside every lym session.


## Issue comment, 2026-11-27

Workaround used: with `lym loom pr-continue` refused in every lym session and GitHub refusing an approval of one's own pull request (merge commits are also disabled on the repository), the orch added `"continue": true` by hand to the run's `_lym/shed/<slug>/pull-request.json`, on the operator's instruction. PR-Review read it within seconds, closed the pull request (`closed_for_continue`) and went on to Finalize. A sanctioned way for the orch to end the review on the operator's word would remove the need for a hand edit.