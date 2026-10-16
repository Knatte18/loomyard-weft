# Plan round 1 — fixer report

## Changes

- F1 (MEDIUM, scope): fixed as proposed.
  Card 1's Intent gains a `Bound (D6)` paragraph after the `readRepositoryRules` paragraph, one sentence per line: the render reads one fixed path in the prime worktree (`Paths.RepoRulesPath`, set from `WorktreePath()`), never a task pair's copy, with no size cap and no new verb or flag;
  the file carries any text into the role file at every delivery and reaches the prime through any landing, including an alignment-approved one;
  the `## This repository` section's own text is the only guard against a contradicting rule.
- F2 (LOW, consistency): fixed as proposed.
  Card 1's Edit entry `internal/orchcli#orchPaths` is replaced by the file path `internal/orchcli/paths.go`, covering the file comment and `orchPaths`; `internal/orchcli#PrimePaths` stays under Uses.
  `lyx loom validate-plan` passes with only the baseline's informational `create-new-unit` finding.

## Disputed

None.

## Deferred

None.
