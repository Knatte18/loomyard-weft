# Hardener campaign on fabricengine, one-repository layout

`harden-fabricengine` · Task · bug, fabric · starting · Worktree-Create

## Goal

Harden `internal/fabricengine` by running it: the package holds the one-repository hub layout from 57015a52, records and board under origin's `refs/lym/` (clone, seed push, one-URL join, refspecs, archive tags), beside the two-repository layout.
This is also the first real run of the hardener recipe, so any defect in the recipe itself is a finding too: report it in the run's report and as a board note.

## Scope

- Campaign package: `internal/fabricengine`, set in `run_settings`.
- Live gate: hub keys `harden-live` tiers 0 and 1, `harden-live-copies` tier 1, `harden-burler-fix` tiers 0 and 1. The package's tests drive git and tmux, no LLM.
- Review rounds capped at 3 for this first run.
- Fixes stay inside `internal/fabricengine` and its tests; a defect found elsewhere becomes a board note.
