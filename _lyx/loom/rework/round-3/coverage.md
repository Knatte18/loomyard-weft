# Rework coverage — PR #321, round 3

- **A. Evidence-gated `accept-audit`, no bare acknowledgement** — cards 51, 52, 53, 54.
  Card 51 adds the suspect-path check against the last batch head, refuses pathless or uncheckable findings toward `run --fresh`, and drops the `accepted` disposition.
  Card 52 runs the check before `run --fresh` archives anything.
  Card 53 re-asserts a suspect-path check in `PersistRecoveryTerminal`.
  Card 54 updates the refusal table.
- **B. Plan writes and rebaseline** — cards 48, 50, 54.
  Card 48 makes a fork write under `PlanDir` a correctness finding.
  Card 50 hashes every plan file when the run records its plan, makes `rebaseline` take `--card NN...`, and refuses an unnamed changed card or a changed `00-overview.md`.
  Card 54 updates the refusal table.
- **C. Every fabric reference is correctness; delete the shell parser** — cards 47, 54.
  Card 47 classes every fabric reference as correctness and deletes `mutatingCommand`, `splitShell`, `readOnlySegment`, the allowlists and their tests.
  Card 54 merges the two fabric-reference rows in the refusal table.
- **D. Parent-write classification gaps** — cards 49, 54.
  Card 49 adds `Geom.ScratchDir` to `runDirs` and makes a write into another worktree of the task repository correctness.
  Card 54 updates the parent-write rows.
