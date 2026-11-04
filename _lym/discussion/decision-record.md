## Goal

```markdown
Two parts, both on a beam's pull request lifecycle, from issues #46 and #44.

## Part 1: reopen a beam from PR-Review (issue #46)

A beam's pawl run cannot take new work once it has left Beam-Await. Suppose the operator flags a beam ready, the run reaches PR-Review, and then a new task should join the beam. Today that takes several untested manual steps:

- `lym loom goto --to Beam-Await` only moves a halted run, but at PR-Review the run is awaiting, so it has to be paused first and resumed afterwards.
- `lym beam ready` writes and commits `_lym/landing/beam-ready.json`, and no `unready` verb exists. While the flag stands, the dobby lands the beam again as soon as nothing works under it.
- The open pull request to the parent stays open with a stale body.
- The pr-reviewer's rewritten `summary.md` in the weft goes stale too.

This came up on beam `duplication-and-import-followups` (NORCE-DrillingAndWells/Models PR #267).

Wanted: one operator verb, for example `lym beam reopen <slug>`, that does all of this in one step:

1. Refuses unless the beam's pawl run is at PR-Review, or at a row after Beam-Await and before Finalize.
2. Stops the pr-reviewer, keeps the pull request open, converts it to a draft and posts a comment saying the beam was reopened.
   The next PR-Review reuses that PR and Beam-Describe refreshes its body, which keeps the PR's link, review comments and history.
   A flag closes the PR instead, for when the reopened work changes the beam's scope enough that a fresh PR reads better.
3. Removes the ready flag.
4. Moves the run back to Beam-Await and leaves it waiting, so the beam can take new members. A later `lym beam ready` lands it again through Parent-Align, Beam-Describe and PR-Review.

## Part 2: name the PR in squash commits and list a beam's sub-PRs (issue #44)

When a task lands through a pull request, the squash commit lym writes carries only the task title, for example `Tool-joint pressure loss in the hydraulics`. Nothing in the commit names the pull request it came from. Landed board entries are pruned, so their `pull_request` field is gone too.

On the same beam the operator wanted the beam's PR description to list every sub-task PR merged into the beam branch. The pr-reviewer could not rebuild that list from git or from lym's records, which named only two of the eleven PRs; the orch listed them by hand with `gh pr list --base <beam-branch> --state merged`.

Wanted:

1. Squash commits end the subject with the PR number the way GitHub's own squash merge does: `Tool-joint pressure loss in the hydraulics (#266)`.
2. Beam-Describe lists every sub-task PR merged into the beam branch, oldest first. Each entry gives the linked PR number, title, merge time and a short summary of what it did. Commits made straight to the beam branch with no PR (including merges from the parent) appear in the same order with a commit link. The final body of PR #267 shows the wanted shape: https://github.com/NORCE-DrillingAndWells/Models/pull/267

With item 1 in place, Beam-Describe reads the PR numbers from the beam's own commit subjects.
```

## Scope

## Decisions

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria
