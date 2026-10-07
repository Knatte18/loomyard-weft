---
round: 1
focus:
  - "Cards 1 and 2: check that no `startup_context` reference survives anywhere (estimator, dynamic program, breakdown, `template.yaml`, `doc.go`, webster state and CLI tests), and that the batch-position start is applied identically in every path that prices a fork."
  - "Cards 2 and 5: check that the position handed to the batchifier (batches before its cards) and the position calibration assigns each fork use the same zero-based or one-based convention."
  - "Card 3: check that the old history flag is gone from code, tests, docs and usage text, and that the one repository flag drives tags, plan commits and webster records through a single constant that tests reference without naming the fabric side."
  - "Cards 8, 9 and 10: check that the card-fabric-reference check shares the audit's location-free patterns rather than copying them, and that it covers every surface the card names (Verify values, verify section, fenced blocks, inline spans) in both plan gates and `validate-plan`."
  - "Card 11: check that `accept-audit --batch` accepts a committed batch only when every fabric-reference command is read-only, and still refuses a batch that writes or runs a mutating command."
  - "Every card: confirm each of its groups' type-specific mechanical checks ran against that group's own targets, and that new doc comments follow the repository's constraints document, including no fixed-column wrapping."
  - "Tests added across the diff: look for tests that duplicate an existing test's coverage, per-helper tests the public surface's test already covers, and added behavior with no test."
---

Round 1 seed from the plan's Card Index and Shared Decisions; no round has been judged yet.
The review range is `main`'s merge base to HEAD, since no rework round exists.
