---
round: 2
focus:
  - "Card order: for each card that changes what `CloneAndWire` or `ReconcileAll` materializes in a worktree, check whether any test or reader that still reads the old per-worktree `board.yaml` is fixed only by a later card, so the earlier card's commit would fail the plan's verify commands."
  - "Hub-wide write in the config CLI card: check that its Intent states what bounds a failed snapshot restore and a process kill between the write and the strict-load validation, given that a later whole-repo board commit stages everything in `_board`."
  - "Seeding of hub-wide config files from the prime worktree's copy: check that the decision record licenses each module seeded, and that the Intent states which source wins when a legacy fold-in and a prime copy both exist and which `Seed` value is reported."
  - "Each new refusal path the config CLI card introduces: check that its message and a test row reaching it are both planned."
  - "Doc updates named by the cards: check that each doc statement a card makes about a header or comment matches what the file contains, and that docs still placing `board.yaml` per worktree are covered by some card."
---

Focus entries are starting points; the full rubric still applies.
