---
round: 3
focus:
  - "In the decision record's D4 on `publish_verify`: settle whether every claim about where a task's edit of `publish_verify` is visible and what it can skip is true, given that `_lyx` in a task worktree is a symlink into the weft repository, landing carries no per-worktree config files to the parent, and Finalize's notice is queued only after the landing."
  - "In D4 and D2: settle whether the decision states which landings run no `publish_verify` and therefore no tmux tier (a parent outside `require_pr_to_base`, where Publish is a no-op and Finalize never runs the key), and whether the stated bound for that case matches `internal/landingshed/doc.go`."
  - "Across D1, D2 and D4: settle whether every sentence about which gate runs the `tmux` tier and which runs the `llm` tier agrees with the D1 table and with where `publish_verify` is set."
  - "Across D6, D7 and the kept `PATTERN-comment-line-breaks` wording that D11 sends the plan writer to: settle whether the rules for a touched line, an untouched neighbour and a sentence end with closing punctuation are mutually consistent, so a fix at a flagged break never forces rewrapping a line the rules call untouched."
  - "In D7's no-base case and D5's statement of which gate catches an unmarked repo-scanning guard: settle whether each stated bound matches the recipe order (Webster, Webster-Burler rounds, Publish, Finalize) and whether a round's own fixer commits are linted anywhere."
---

Focus entries target the open design and consistency questions of the latest round, restated from the decision record and the sources they depend on.
