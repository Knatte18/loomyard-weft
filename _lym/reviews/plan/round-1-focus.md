---
round: 1
focus:
  - "Fidelity: read _lym/discussion/decision-record.md and settle whether every Decision and Constraint (D1 to D9) is carried by a card, and whether the Shared Decision `out-of-part` scope exclusions (hold, registry, catch-up, adopt) are licensed by it."
  - "Card order: cards 3 to 6 build the push and takeover pieces that cards 7 to 13 consume; settle whether any card uses a symbol (`PairPush`, `PushRun`, `HandOver`, `IsPushOwed`, `MachineNames`) that only a later card creates."
  - "Reachability: for each refusal or guard branch the cards prescribe (takeover read on a rejected push, `teardown_lock` owed push, the code-tip grace and the owner's tip guard), settle whether a named input and a named test reach it and no upstream check rejects that input."
  - "Attack surface: settle whether cards 5, 6, 11 and 12, which add a hand-over that ends processes and removes a pair, and a give-up after the grace, state in `Intent` what they can skip and what bounds it."
  - "Board entry: settle whether card 10 (board done) or any other card changes this task's own board entry body or scope, or files or promotes a task."
  - "Edit cards: settle whether `Edit` cards that name specific symbols in `Intent` list member glyphs, and carry the re-sign arrow where parameters or results change (`PushPairAnchored`, `coalescePushSides`, `ReadRun`)."
  - "Test economy: settle whether new tests in cards 3, 4, 7 and 8 duplicate behavior an existing test covers, and whether any card tests static prose in PATTERN files (card 14)."
  - "PATTERN fit: check each card against the repository's `PATTERN.md` entries, including `PATTERN-fabric-boundary` and `PATTERN-test-only-code`, which the plan cites."
  - "Verify lines: settle whether any card's Verify line names a raw test command instead of one `lym gate test --wait` call."
---

The plan carries lens-review files from the writer's own pre-review; they are context only and set no verdict.
