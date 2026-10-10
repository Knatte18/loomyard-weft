---
round: 1
focus:
  - "Fidelity: check each Decision and Constraint in `_lyx/discussion/decision-record.md` (D1-D6 and the gaps #513, #514, #518) is carried by some card, and that Shared Decisions (Card 4 pinning the set the walk reports, Card 5 adding rows only) are licensed by the record and not new scope."
  - "Test economy: Cards 1-3, 5, 6, 7 and 8 plan rows or assertions in existing tables; confirm each names the table it extends and that no card adds a top-level test where an existing one covers the behavior. Card 4's new `--json` pin is the only one the overview licenses; check its package's test-budget row is listed under `Edit:`."
  - "Verify coverage: every package a card targets (classifier, websterengine, shuttleengine, the Merriam-base caller packages, the command tree for Card 4) is run by the `## verify:` section, including `internal/shuttleengine` and any build-tagged tests the cards touch; check the section order against the plan template."
  - "Edit-card glyphs: for each `Edit` card whose `Intent` names specific functions, rows or symbols, settle whether the targets list member glyphs rather than only Go file paths, and whether the re-sign arrow is present where the `Intent` changes parameters, results or receivers (Card 2 changes the classifier's word type; Card 6 changes `MerriamBase`'s error)."
  - "Attack-surface: Cards 2 and 3 widen what the read-only classifier admits (quoted, globbed, tilde, `2>/dev/null`, `sed -n`, trailing `--json`), and Card 5 adds git reader forms; settle whether each `Intent` states what is now let through and what bounds it."
  - "Card order and ImpactSummary: Cards 2, 3 and 5 share the classifier and Card 4 depends on Card 3's bound; settle whether any card uses something a later card creates, and whether each `Edit` and `Delete` card carries a one-line blast-radius conclusion rather than a restatement of `Intent`."
---

Seed focus only; every entry names where to look and leaves the verdict to the reviewer.
