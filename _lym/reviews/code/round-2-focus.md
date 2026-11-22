---
round: 2
focus:
  - "In internal/burlerengine/cluster.go, check whether any git command that changes a remote's URL, push URL or config (set-url with or without --push, remote remove or rename, --git-dir or --work-tree, worktrees sharing a source repository's config) can leave a remote recorded as leading into a fork repository while git pushes elsewhere."
  - "In internal/burlerengine/cluster.go, check whether any spelling of a config override (-c, --config-env, include.path or includeIf keys, clone -c) can redirect a push without setting the redirect flag, and whether clone operands named like config keys wrongly set it."
  - "In internal/burlerengine/cluster.go#leadsInto, check which directory a relative push destination or remote URL is resolved against, compared with where git resolves it from."
  - "In internal/burlerengine/cluster.go#forkViolation and the fix-round halt text, check that every claim the halt makes about the target's repository is one the audit can know, for pushes from a fork repository to an unplaced destination and when a non-git violation is reported first."
  - "In internal/shedadapters/burlerfix.go, check what the stored fixer rerun record holds after a failure exit that archived no report, after two died attempts, and after a round's review, report and acceptance are retired."
  - "In internal/shedadapters/burlerfix_test.go and internal/burlerengine/cluster_test.go, check that the stored rerun record's reason and report, the cancellation reason, and each new guard branch are asserted by some test."
---

Each entry names a place to look and the question to settle; none caps severity.
