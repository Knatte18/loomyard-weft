# Halt a segment after review rounds that change nothing in the tree

`stalemate-no-tree-change` · Note · feature, burler, shed

## Seen (orch, 2026-11-17)

A run's escalation cause `circling` (`internal/loomcli/circling.go`) is an LLM's judgment.
A round whose fix changes nothing in the warp tree is a stalemate a Go check can see without one.
Source: the survey F17 (`orchestrator-survey-ideas`), from ralphex's `--review-patience=N`, which stops after N rounds with no commit and no tree change.

## Idea

- The Bouncer counts review rounds whose fixer left the warp tree unchanged.
- After a set number of them in a row, it halts the segment with a named stalemate cause before the bounce budget runs out.
