---
round: 4
focus:
  - "D4 in _lyx/discussion/decision-record.md: for every key of every module, hub-wide and per-worktree, settle whether the category test that D4 states yields exactly the LocalKeys lists as written. A key that gates or shapes a landing into the shared parent branch, or switches off an agent guard or review, must not be open to the local layer, and each Decision that opens such a key must state what it can skip and what bounds it."
  - "D4 Rationale and R3: settle whether the claim that the denied keys persist in a pair only through a tracked file holds, given that configengine.load expands ${env:...} markers via envsource.Build(baseDir) (internal/configengine/config.go), which reads the git-ignored per-worktree <anchor>/.env before the process environment, and whether R3's stated bound on what the local layer gains is true."
  - "D6 and its acceptance criterion: settle whether the strip states that it compares unexpanded nodes, so a literal in a worktree file is never stripped because it equals the walk's expansion of a ${env:...} marker below, and the reverse; and whether an acceptance row covers that case."
  - "D6 claim that migration never changes what a worktree resolves: settle whether it holds for keys the seeded default adds beneath a worktree file that lacks them, for example a seed-only alias or lens the prime holds, since D1 lets no layer delete a key below and the strip only removes keys."
---

Each entry is a question to settle against the current text, not a finding to confirm.
