---
round: 1
focus:
  - "Cards 4, 7 and 8 each edit the same `internal/webstercli` verb files and `runlevel.go`: settle whether the split is one reviewable unit per card or the same call-site change is made three times (design: batching and sequencing)."
  - "Card 4 edits `internal/batcher/registry_test.go`, and card 6 deletes it: settle whether card 4's edit is wasted work or the plan's order needs it."
  - "Card 7 removes the `cycles` key and the cycle warnings, and cards 7 and 9 change a refusal's trigger: settle whether each card's `**Intent:**` states what the change lets through and what bounds it."
  - "Card 8's `ExecutionBatches` falls back to identity for a state without a partition, and card 9 does the same for the tail: settle whether each card states the bound on that fallback."
  - "Decisions 6 and 8 name an `alone_above`/`budget` relationship and a `max_cards` floor of 2: settle whether cards 5 and 6 carry each of those decisions and constraints, with no scope outside the decision record."
  - "Card 10 runs a calibration command and may end without a commit: settle whether its flags and the `10`-card and spread thresholds are licensed by the decision record's Auto-mode assumptions."
  - "`## verify:` runs `-tags integration` over a fixed package list: settle whether every package a card targets with `integration`-tagged tests is in it, notably `internal/batcher`, `internal/planparser` and `internal/gitrepo`."
  - "Test economy in cards 2, 3 and 5: settle whether any planned test repeats behavior an existing test already covers, such as `TestHistoryReads`, `TestActive` or the identity tests, or tests a single symbol rather than the public surface."
  - "Granularity of card 2: settle whether `SizeSource`, `Weights`, `SegmentCost`, `DiskSizes` and `ProfileWeights` each earn a separate `Create` entry, or are one surface's helpers that belong in a surface card."
---

Targeted at where the plan's structure is most likely to be wrong: repeated edits across cards 4, 7 and 8, `verify:` coverage, and the attack-surface statements on cards that remove or relax a guard.
Every entry was checked against the `Do not flag` list; none re-derives a gate check, a derived dependency edge or an `ImpactSummary` on a type that does not require one.
