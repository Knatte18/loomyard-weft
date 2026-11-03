# Land a pull-request run on GitHub after the review is ended

Ending the PR-Review through the agent used to close the pull request, and Finalize then squash-merged locally and pushed the parent directly.
A parent branch whose GitHub ruleset requires a pull request refuses that push, so the run blocked at Finalize with local `main` ahead of origin and no way forward in lym.

With a pull request wanted, the landing is now always a squash merge on GitHub, never a local merge pushed to the parent.

Closes #20

## What changes

- Ending the review through `lym loom pr-continue` leaves the pull request as GitHub has it and records the operator's ending in the pull-request record.
  The `continueComment` close is gone.
- Finalize merges the pull request on GitHub as it does for any open one.
  When the record carries the operator's ending and the pull request is closed, for example one closed by an older build, Finalize reopens it first, since GitHub squash-merges only an open pull request.
  A closed pull request without the operator's ending still halts, and its way forward now names `lym batten cancel`.
- Finalize's parent side takes the merged result from origin through the existing fast-forward, so local and origin parent do not diverge.
- lym reads the parent branch's rules (`GET repos/{owner}/{repo}/rules/branches/{branch}`).
  A branch with a `pull_request` rule counts as a required parent whatever beam.yaml and the entry say, so lym never chooses a direct push to it.
- A refused merge no longer points the operator at `lym loom goto PR-Review` to close the pull request and land locally; its way forward is a second approval, a settled check or conflict, or `lym batten cancel`.

## Design decisions

- `ResolvePullRequest` takes a new `parentRuleRequired` argument.
  `resolveWithParentRule` in `internal/landingshed/pullrequestmode.go` is the one place both PR-Review and Finalize ask it, so the two rows cannot disagree.
- The rules are read only when beam.yaml and the entry give a no for a task branch with commits and an origin a pull request could be opened on.
  Landings already sent through a pull request, and ones that never reach GitHub, make no extra call.
- Failure of the rules read:
  - A cancelled context or a transient failure is an error the driver re-steps.
  - With a pull request recorded, any other failure halts, an unusable token excepted.
    Reading it as no rule would close the recorded pull request and land by the direct push GitHub refuses.
  - With no record, any other failure, an unusable token included, logs a Warn and reads as no rule.
- The record field `ClosedForContinue` is renamed `ReviewEnded` in Go.
  Its JSON key stays `closed_for_continue`, so a record an older build wrote reads the same.
- `pullrequest.API` gains `Reopen` and `RequiresPullRequest`.
  `RequiresPullRequest` reads every page of the active rules.
  `PATTERN-github-auth` now lists reopen among the actions only PR-Review, Finalize and pair teardown take.
- The stencil `landing-template-pr-review`, `docs/overview.md`, and the `landingshed` and `pullrequest` package docs describe the new flow.

## To check by hand

- Not run against GitHub: end a review through the PR-Review agent on a repository whose parent branch has a ruleset requiring a pull request (squash only, linear history), and confirm Finalize merges the open pull request and `main` fast-forwards to origin.
- Not run against GitHub: take a run whose pull request an older build closed with the operator's ending, resume it at Finalize, and confirm the reopen and merge.
- Bound, as documented: a rule the read misses, and branch protection set outside repository rules, still end in a direct push GitHub refuses.
