---
round: 1
focus:
  - "Card 10 (`pattern-testing-consolidation`): read the diff of `PATTERN.md` and `pattern/` and settle whether every retired entry ID has no remaining reference anywhere in the repo (code comments, docs, tests, other `pattern/` files), whether the deleted background files leave dangling links, and whether each `pattern/` file is named by exactly one entry."
  - "Card 13 (`retag-smoke`): settle whether every per-package LLM-binary lookup the card lists was replaced by `llmkit.Claude` with the override variable that copy honoured, and whether any local copy or `smoke` build tag survives in the diff or its docs."
  - "Cards 13 and 14 (retags): settle whether each retagged file's tier matches the card's detection rule (a file is `tmux` only when its tests start a tmux server), and whether shared helper files carry the tag disjunction the Shared Decisions require."
  - "Card 15 and card 12 (tier guards): settle whether the LLM-spawn guard and the tier guards cover the `tmux` and `llm` tags as the cards specify, and whether the `llmkit` `os/exec` exemption is bounded as the card says."
  - "Cards 3-6 (timing seams): settle whether each shortened wait is a per-instance struct field or parameter rather than a package-level `var`, whether production passes today's value, and whether the newly parallel tests share no state that makes `t.Parallel` unsafe."
  - "Cards 17-19 (socket cleanup): settle whether every path that kills or tears down a tmux server removes its socket file, and whether any ad hoc tmux key in tests remains outside `tmuxkit` after card 18."
  - "Cards 20-21 (`testtiming` redundancy mode and committed report): settle whether the new mode's behavior has test coverage, and whether new tests duplicate the coverage of an existing `testtiming` test."
  - "All new or changed Go doc comments and the `doc.go` updates: settle compliance with the repository's own `CLAUDE.md` and `PATTERN.md` comment and semantic-line-break rules."
  - "Every card's own groups: confirm the type-specific mechanical checks (Rename script plus grep, `assert-no-callers` for Delete) ran and passed against each group's own targets."
---

Seeded from the plan's Card Index and Shared Decisions only; the diff has not been read in depth.
Entries name where to look, not verdicts.
