# Name the PR in squash commits and list a beam's sub-PRs in Beam-Describe

`squash-names-pr` · Note · enhancement, landing, pawl

Imported from [issue #44](https://github.com/Knatte18/loomyard/issues/44).

When a task lands through a pull request, the squash commit lym writes carries only the task title, for example `Tool-joint pressure loss in the hydraulics`. Nothing in the commit names the pull request it came from. Landed board entries are pruned, so their `pull_request` field is gone too.

On beam `duplication-and-import-followups` (NORCE-DrillingAndWells/Models PR #267), the operator wanted the beam's PR description to list every sub-task PR merged into the beam branch. The pr-reviewer could not rebuild that list from git or from lym's records, which named only two of the eleven PRs. The orch had to list them by hand with `gh pr list --base <beam-branch> --state merged`.

Wanted:

1. Squash commits end the subject with the PR number the way GitHub's own squash merge does: `Tool-joint pressure loss in the hydraulics (#266)`.
2. Beam-Describe lists every sub-task PR merged into the beam branch, oldest first. Each entry gives the linked PR number, title, merge time and a short summary of what it did. Commits made straight to the beam branch with no PR (including merges from the parent) appear in the same order with a commit link. The final body of PR #267 shows the wanted shape: https://github.com/NORCE-DrillingAndWells/Models/pull/267

With item 1 in place, Beam-Describe can read the PR numbers from the beam's own commit subjects. Without it, Beam-Describe can query GitHub for PRs whose base is the beam branch.