# Pull-request landings merge on GitHub and honor the parent's PR rule

## Problem

With `pull_request` on, ending the review through the PR-Review agent (`lym loom pr-continue`) made the PR-Review row close the pull request.
Finalize then squash-merged the task into the local parent and pushed the parent directly.
A parent whose GitHub ruleset requires a pull request refuses that push.
The run blocked at Finalize with the local parent one commit ahead of origin and no lym way forward.

## What changes

- **The operator's ending no longer closes the pull request.**
  On the continue flag at a clean, gated head, the PR-Review row records the ending in the run's pull-request record and ends Done.
  The pull request stays open, and Finalize merges it on GitHub as it does any open one: push the task branch, merge with the pushed head as the expected head, then fetch and fast-forward the parent pair.
  Local and origin parent never diverge.
- **Finalize reopens a closed pull request that carries the operator's ending.**
  GitHub merges only an open pull request, so a closed one whose record carries the ending is reopened and then merged on GitHub.
  This covers a pull request someone closed on GitHub while the continue flag stood, and a record left by an older build.
  A failed reopen is judged as any GitHub failure, with GitHub's message and `lym loom resume` as the way forward.
  A closed pull request without the ending still halts as before.
- **GitHub's rules for the parent branch count as a required parent.**
  `pullrequest.API` gains `RequiresPullRequest`, which reads `GET repos/{owner}/{repo}/rules/branches/{branch}` across every page and reports whether any active rule is a `pull_request` rule.
  `ResolvePullRequest` takes the answer as `parentRuleRequired`, and treats it like `beam.yaml`'s `required` whatever `beam.yaml` and the entry field say.
  Both PR-Review and Finalize ask through `resolveWithParentRule`.
  It reads the rules only when `beam.yaml` and the field give a no for a task branch with commits and an origin a pull request could be opened on, so a landing already going through a pull request, or one that never reaches GitHub, makes no extra call.
  A transient read failure is an error the driver re-steps.
  Any other failure, an unusable token included, logs a Warn and reads as no rule, so a hub without a GitHub token still lands as `beam.yaml` says.
- **`pullrequest.API` gains `Reopen`**, a PATCH of the state back to `open`.
- **Way-forward text.**
  Finalize's merge-refusal halt no longer offers "ask the PR-Review agent to continue, which closes the pull request and lands the task locally"; it names a second person's approval or a settled check or conflict, then `lym loom resume`, or `lym batten cancel <slug>`.
  `lym loom pr-continue`'s help and note, the PR-Review stencil, `docs/overview.md`, the package docs and `PATTERN.md` (`PATTERN-pull-request-mode`, `PATTERN-github-auth`) now say the ending leaves the pull request for Finalize to merge on GitHub.

## Design decisions

- **Keep the pull request open, not reopen it.**
  The issue allowed either approach and called keeping it open the simpler one.
  The reopen path exists only for a pull request that ends up closed anyway.
- **The record field is renamed `ReviewEnded` but keeps its JSON key `closed_for_continue`.**
  A record an older build wrote mid-run then reads as an ended review, and Finalize reopens and merges its closed pull request instead of halting.
- **The rule read is lazy and soft.**
  The rule only matters when lym would otherwise push directly, so the read happens only on that path.
  An unreadable rule falls back to `beam.yaml`.
  Halting there would break every token-less hub with a GitHub origin.
- **Push skip and a missing upstream skip the read.**
  With either, Finalize pushes nothing to origin, so no rule can refuse the landing.

## Bounds

Branch protection configured outside repository rules (classic protected-branch settings) is not read, and a rule the read misses still ends in a direct push GitHub refuses, as before.

## Tests

- `internal/pullrequest`: `Reopen` sends the state PATCH; `RequiresPullRequest` reads a `pull_request` rule on a second page, answers false for other rules and for none, and returns a failure.
- `internal/landingshed`:
  - `ResolvePullRequest` under a parent rule: opens, warns on a field off, halts on a missing origin fact with no cancel clause, and stays a no on an empty task branch.
  - PR-Review opens a pull request for an off parent whose rule requires one, reads an unreadable rule as none, and returns a transient read failure as an error.
  - Finalize merges on GitHub under an off parent with a rule, returns a transient read failure as an error stop, reopens and merges a closed pull request with the ending, and halts on a failed reopen.
  - The wait's continue rows assert no `Close` call and the recorded ending.
