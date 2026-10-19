---
round: 1
focus:
  - "Fidelity: check that every Decision D1-D7 and each Constraint in the decision record is carried by some card, including the toolfloor audit in `doc.go`, the `docs/overview.md` module tree entry, the `pattern/PATTERN-leaf-packages.md` listing, the `spawnobservability_test.go` allowlist removal, and the Windows-only no-probe test file."
  - "Card order: cards 4, 5, 6 and 7 use toolfloor and gitexec's exported check; settle whether each depends only on earlier cards, and whether card 1 really precedes everything that needs a runnable test package."
  - "Attack-surface: for the floor checks, the cache bypass for hook-driven `TmuxCmd`, the unparseable-line admit, and the exported check, settle whether each card's `**Intent:**` states what it lets through and what bounds it."
  - "Reachability: for each refusal a card prescribes (retry-bound exhaustion in lyxtemp, empty-from-start commit in CherryPick, floor errors, `--skip` failure), settle whether a named input and a named test reach it and whether an upstream check rejects that input first."
  - "Verify coverage: settle whether every package any card targets, including `tools/` and any test kit, has its tests run by the `## verify:` section, hermetic build-tagged tests included, and whether the tiers are in the order given."
  - "Edit-card glyphs: for Edit cards whose targets are only non-test Go file paths while `Intent` names symbols, and for Intents that change a signature, settle whether member glyphs and the re-sign arrow are present."
  - "Test economy and granularity: settle whether cards 8, 9 and 10 plan new tests for behaviour an existing test covers, and whether card 2 splits symbols per card that belong together."
  - "Board entry: settle whether any card edits this task's own board entry body or scope."
---

Seed focus drawn from the overview and the decision record; no prior round exists.
