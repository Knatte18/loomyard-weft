# Ending the PR review closes the pull request and lands by direct push, which a PR-only branch refuses

`pr-review-end-direct-push` · Task · bug, landing, git · running · Describe

Imported from [issue #20](https://github.com/Knatte18/loomyard/issues/20).

## What happened

Hub NORCE-DrillingAndWells/Models, beam `loomyard-setup` landing into `main`, prime beam.yaml `pull_request: "on"`.
PR-Review opened PR #254 (loomyard-setup -> main).
The review was ended through the PR-Review agent.
lym then closed #254 with the comment "the operator ended the review through the PR-Review agent; lym closes this pull request and the run lands through Finalize", and Finalize squash-merged locally into `main` and pushed directly.

`main` carries a GitHub ruleset requiring a pull request (squash only) and linear history, so origin refused the push:

```
push rejected: ! [remote rejected] main -> main (push declined due to repository rule violations)
```

The run blocked at Finalize with local `main` one commit ahead of origin and no lym way forward: retrying the push fails on the same rule.

## Expected (operator, 2026-11-03)

With `pull_request` on, the landing is always a squash merge on GitHub into the parent branch, never a local squash merge pushed to the parent.

- Ending the review through the PR-Review agent may close the review, but Finalize still lands by squash-merging the pull request on GitHub, as it does for an open one.
  GitHub squash-merges only through a pull request, so if lym closes the pull request at the end of the review, Finalize reopens it, or opens a new one, and merges that; keeping it open is the simpler way.
- Finalize's parent side then takes the merged result from origin, as it does after any GitHub merge, so local and origin parent never diverge.
- lym reads the target branch's rules (`GET repos/{owner}/{repo}/rules/branches/{branch}`) and never chooses a direct push to a branch that requires a pull request; such a branch then counts as `pull_request` on whatever beam.yaml says.

Code: `internal/landingshed/prreview.go` closes the pull request with `continueComment`; `internal/landingshed/finalize_pr.go` holds Finalize's merge on GitHub and its decision about a closed pull request.

## Workaround used

Reopen the PR, squash-merge it on GitHub by hand, reset local main to origin/main, cancel the run.
