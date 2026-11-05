# Fixer-report, code round 2

## Changes

- F1 (LOW, consistency): fixed as proposed. `PATTERN.md:13`, the `PATTERN-durable-vs-ephemeral-state` line, now reads "bar heddle's rendered files and `.heddle` at a warp worktree's anchor".
  Commit a292ca4b trimmed "never-tracked" to "untracked" and "variable block" to "variables" to stay under the headroom line.
  The self-check showed the line fits without those trims (the overview is 15560 bytes against a 15564-byte headroom line), so commit "restore never-tracked and variable block" put the original words back.
  `lym gate test ./internal/pattern` passes.

## Self-check

- fix-code-correctness: LOW, the trims of "never-tracked" and "variable block" drop meaning the review never asked to drop. Fixed with my own change: both phrases restored, the line fits without the trims.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
