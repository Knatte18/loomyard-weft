---
round: 2
focus:
  - "Card granularity around the segment-settings and wiring cards: does each public symbol reused across packages (`shedrecipe.SegmentSettings`, `envkit.TemplateSetting`) sit on a card of its own or on a card with the surface it serves, and does `TestFullEnv_FillsEverySeam` stay green at every card boundary?"
  - "Card order after any card insert or removal: does every `plan:` Use point at a symbol created by an earlier card, and does the Card Index in `00-overview.md` match the card files?"
  - "The `verify:` section of `00-overview.md`: does every package a card targets, `internal/battencli` and `internal/battenrecipe` included, have its tests run by a `lyx gate test` line, in the hub's tier order?"
  - "Attack-surface statement on the card that adds the `_setting` recipe keys: does its `**Intent:**` say what a row can now read and what bounds it?"
  - "Refusal reachability on the setting and budget cards: for each prescribed refusal or guard branch, does a named input and a named test reach it, and does the asserted text match what the producing function can emit (`configString`, `configengine.Load` propagated errors, `segmentBudgets` and `checkSegmentBudgets` way forward)?"
---

The focus targets the areas the plan's cards changed in round 1; it lists no pre-judged verdicts.
