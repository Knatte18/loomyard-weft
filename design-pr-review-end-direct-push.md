# Ending the PR review closes the pull request and lands by direct push, which a PR-only branch refuses

`pr-review-end-direct-push` · Task · bug, landing, git · draft

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

## Expected

With `pull_request` on, ending the review should let Finalize merge the open pull request on GitHub (squash), as it does for an open PR, not close it and land by direct push.
At minimum, lym should read the target branch's rules (`GET repos/{owner}/{repo}/rules/branches/{branch}`) and never choose a direct push to a branch that requires a pull request; a pull_request mode `required` on the prime would then be implied by the remote.

## Workaround used

Reopen the PR, squash-merge it on GitHub by hand, reset local main to origin/main, cancel the run.