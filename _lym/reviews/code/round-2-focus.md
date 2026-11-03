---
round: 2
focus:
  - "Test coverage of the operator-only removal: the diff sets PruneUnknown at the hub worktree-file call site in internal/configcli/configcli.go and the default-layer call site in internal/configcli/hubwide.go. Settle whether a test fails when either flag is dropped, by seeding a key the template lacks and no retired key covers and asserting `lym config reconcile` removes it."
  - "Test coverage of the automatic callers: the hubreconcile walk worktree reconcile in internal/hubreconcile/walk.go, the clone paths in internal/hubreconcile/clone.go, and the fabric reconcile call in fabric.go leave PruneUnknown unset. Settle whether a test would fail if one set it, for a worktree file (not only the board fabric.yaml)."
  - "pruneSparse in internal/configsync/configsync.go: when a retired key is dropped and another key is preserved, settle whether the returned bytes and the Removed and Preserved lists come from one consistent pass, with no empty mapping left behind and no overlap between Removed and Preserved, including nested keys."
  - "Wording of the config-layers rule in PATTERN.md and pattern/PATTERN-config-layers.md, and the hubreconcile doc.go paragraph: settle whether each claim about who removes, keeps or reports a key the template lacks matches the code, including the standalone worktree base (reconcileBaseWorktreeFile) and which callers actually log or read Result.Preserved."
---

Focus targets the five round 1 areas so a fix is checked by a fresh read of the code, not by a prior finding.
