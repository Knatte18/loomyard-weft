---
round: 1
focus:
  - "Card 1: does the `kickstart_pack_per_card` key default to true in both the struct default and the written/loaded `webster.yaml` path, and does an absent key in an existing file load as true?"
  - "Card 2: does `lyx loom card-pack <card>` stay read-only and resolve against the tree as it stands at call time, with an unknown or malformed card argument failing clearly?"
  - "Card 3: do the file and inline layouts, the pack-off prompt and the recovery prompt keep their bytes, and does the begin-batch trace line record `pack_layout=card` only when the switch is on?"
  - "Card 3: does each `card-pack` call log a `webster: card pack` line, and does the fork prompt order the steps as plan card files first, then the card's pack before its first edit?"
  - "Card 4: does `tools/forkaudit` read the card arm from card-pack results and `webster: card pack` trace lines without changing its existing arms' output?"
  - "Test economy: do the new tests for the switch default, the verb and the layout duplicate coverage an existing test already gives, and is any added behavior left uncovered?"
---

Focus lists where to look per card; none caps severity.
