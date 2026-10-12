---
round: 1
focus:
  - "Card 1, `internal/lyxtemp`: settle whether the sweep's lock guard cannot remove a live owner's run directory, and whether the inherited-run-directory rule (a system temp directory that is already a run directory is reused as the root) holds in the code."
  - "Cards 2, 3, 5, 6: settle whether every run directory a process creates is removed after its run, and whether card 6 drops the per-package `TMPDIR` and attributes leftovers by the inherited marker as the plan states."
  - "Cards 7-11: settle whether every test package calls `tmuxkit.Main` from a `TestMain`, and whether the renamed guard keeps its old coverage; check the new tests against `PATTERN-test-economy` for duplicates of an existing test."
  - "Card 12: settle whether the bare-temp scan catches `os.MkdirTemp`, `os.CreateTemp` and `os.TempDir` uses outside `lyxtemp`, and whether its PATTERN entry landed in the same commit."
  - "Cards 13, 16, 18, 19, 20: settle whether each cleanup computes its candidates identically in dry-run and apply mode, creates no lock file and removes nothing in a dry run, and holds every refusal in the module function rather than the verb."
  - "Cards 1, 2, 3, 12, 13, 15, 16, 18, 19, 20: settle whether each card adding a top-level test edited its row in `cmd/lyx/testdata/test-budget.yaml`, and whether the docs the card touches (`doc.go`, `docs/overview.md`) changed in the same commit."
  - "Cards with Rename or Delete groups: confirm each group's own mechanical check ran against that group's targets."
---

Seed focus from the plan's card index and shared decisions; no round has been judged yet.
