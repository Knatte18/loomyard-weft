---
round: 2
focus:
  - "Import allowlists: for every card that makes a production package gain a new import, check whether that package pins its imports with scankit.AssertImportAllowlistNoStale (an *AllowedImports list in its seam_enforcement_test.go), and whether the card lists the allowlist among its Edit targets. Settle whether any card would fail its own package test."
  - "Orch watcher restarts: for the orch pause card's per-entry tick state, which is held in memory, settle whether a watcher started after a live hub entry's grace end (for example the one lyx orch start spawns for a fresh orch) can interrupt a busy orch that the entry does not own, against the decision that an orch started under a hub entry is not interrupted, and whether a test row reaches that case."
  - "Per-run network hold: settle whether network operations run in the prime on behalf of a pair (batten's Worktree-Create and Land rows) are held by a per-run pause entry for that slug, given the network gate takes its slug from the process location."
---

Focus targets the gating design findings and their neighbours; each entry asks a question and carries no verdict.
