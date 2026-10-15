---
round: 1
focus:
  - "internal/sheddriver imports: does any file import lyxcwd, loomengine, loomshed or an internal/*cli package, contrary to the sheddriver-surface decision?"
  - "Moved files (cards 1, 3, 4, 6): were they moved with git mv under their original base names, except loomengine/tackler.go becoming tacklerrecords.go, and do callers reach moved code only through the declared handles?"
  - "Settle records per run (card 3): does every reader and writer use the per-run shedrun path, with no leftover reference to the old location?"
  - "Override resolution (cards 7, 8): does an empty module value inherit shed.yaml, and does a set tackler_max_spawns of 0 still turn the tackler off (*int, not 0 as unset)?"
  - "Loom override migration (card 9): is it one-time and idempotent, and does the comment refresh answer migration-comments-stale-on-pinned-key-hubs?"
  - "Batten driving (cards 11-13): does lyx batten run drive its run in-process through the sheddriver body, send one orch notice per stop episode, and leave every default driver_kind at llm?"
  - "Test economy: do new tests in internal/sheddriver and internal/battencli duplicate existing coverage, and do the test-budget.yaml rows match the plan's decision?"
  - "Docs and PATTERN (cards 10, 14): do PATTERN-shed-driver, its seam test, doc.go and docs/overview.md match the code that landed?"
---

Seed focus derived from the plan's Card Index and Shared Decisions only; no diff has been read.
