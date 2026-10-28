---
round: 2
focus:
  - "internal/burlerengine/cluster.go, the git branch read-only tables (gitBranchReadOptions, gitBranchReadShortOptions): settle whether any option that git treats as a modifier of branch creation, such as -v, --verbose or -a, is marked list mode, so that `git branch <option> <name>` would pass the fork audit as read-only although git creates the branch."
  - "internal/burlerengine/cluster_test.go, TestFirstMutatingGitCommand: settle whether the rows pin the separate-word value skip for value-taking options (--format, --points-at, --sort) and the trailing double-dash branch of gitBranchReadsOnly, for both an operand before and after `--`."
---

The focus names where to look; it states no verdict.
