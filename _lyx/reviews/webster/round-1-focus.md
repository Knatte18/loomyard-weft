---
round: 1
focus:
  - "Per-card mechanical check: for each card with several groups (Rename, Delete, Move), confirm the type-specific check ran against every group's own targets, not only the first label; start with cards 17 to 22, the reclassification cards, and cards 28 to 30, the spawn-removal cards."
  - "Test economy: new tests in cards 5 to 10, 16, 23 to 26 and 31 to 32 against existing coverage; name the covering test for any duplicate, and any per-helper test the public surface already covers."
  - "Behavior without coverage: the reed stale-holder probe in card 5, the leftover failure in card 7, the socket check in card 10, the `GIT_DIR` and `GIT_WORK_TREE` drop in card 27, and the `LYX_PREBUILT_LYX` strip and deny in cards 13 and 14; settle whether each added behavior has a test."
  - "Decision conformance: the hub-fixture-classification decision (shared when every access is a read, otherwise copy, otherwise `NewHub`) against the actual classification in cards 17 to 22, and the rev-parse-survivors list against the spawns that remain after cards 28 to 30."
  - "Comment-convention compliance on new doc comments, against the repository's constraints document, with `PATTERN.md` outranking inferred style."
---

Seed focus only; the base diff review still applies in full.
