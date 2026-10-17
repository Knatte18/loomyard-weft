# Rewrite C13 stacked half for window-per-strand, commit the creel re-test report

`creel-c13-stacked-rewrite` · Task · enhancement, creel

## Goal

A docs-only change in two parts.
Operator decision 2026-10-17: run as darn with a `sonnet[medium]` writer and no review perch.

## 1. Rewrite C13's stacked half

The board entry `reed-add-parentless-placement` removed the stacked layout and `collapsed_rows`, refuses `below-parent` and `top`, and moves a strand pane left in the shell window into its own window.
C13 in `tools/sandbox/SANDBOX-CREEL-SUITE.md` still describes its stacked 3-row half against the old layout: rewrite that half against the window-per-strand layout, reading `internal/reed`'s package doc for how strands are placed now.
Its hand-added half (a plain `lyx reed add --cmd claude` strand receiving its reminder) stays as is.

## 2. Commit the creel stage 1 re-test report

Copy `/home/knatte/Code/loomyard-LYXHUB/loomyard/.scratch/creel-retest-2026-10-17.md` (in the hub prime, outside this worktree; read it, never edit or move it) verbatim to `docs/research/creel-audit-2026-10-17/report.md` in this worktree.
It is a measurement report of one run on `58a7feec7`, so it stays as written.

## Done when

Both files are committed in one commit; no code changes.
