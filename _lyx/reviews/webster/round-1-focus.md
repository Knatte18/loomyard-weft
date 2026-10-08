---
round: 1
focus:
  - "Cards 7, 8 and 9 (`readRoundUsage` and its consumers): does every reader take the fan and lenses a round ran from `round-<N>-usage.yaml` alone, and does a missing or unreadable record leave exclusions ineligible rather than dropping lenses?"
  - "Card 8 and card 10: are exclusions accumulated by Go across the live generation only, and does the judge prompt ask for exclusions from the judge alone, with the permanence and eligibility rules the plan states?"
  - "Card 3 against card 1: when a round's exclusion would empty the fan, does the fallback to the held exclusions apply, and does per-name fan and lens resolution over the embedded template match each card's group targets?"
  - "Cards 4, 5 and 6: do the shuttle usage fields stay provider-neutral, does the Claude reader count tokens from forks' transcripts as well as the main one, and do both halves' usage and times reach the round record for solo and fanned rounds?"
  - "Cards 11, 12 and 13: does each segment's fan (`discussion_fan`, `plan_fan`) and `fan_review` reach both of that segment's rows, and is an unset fan identical in behavior to before the change?"
  - "Card 14: does the rewritten gate-quiescence guard rest on shuttle's fork wait as the plan states, and is the Webster-Burler recipe comment correct against the recipe's actual rows?"
  - "Per-card mechanical checks: for every card group with a `Rename` or `Delete` type, confirm its mechanical check ran against each of that group's own targets."
  - "Test economy: for each new test, check whether an existing test already covers the behavior and whether each new behavior has coverage."
---

Seed focus drawn from the plan's card index and its shared decision `usage-record-holds-lenses-run`.
