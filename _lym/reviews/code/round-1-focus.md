---
round: 1
focus:
  - "internal/haltnote/haltnote.go Render and parseRow, internal/loomcli/halt.go: `shedengine.HistoryEntry.RoutedTo` is also set on reroute entries, not only on a routine Stuck. Settle whether the `routed to` suffix is right for every outcome that carries it, and whether the doc comments saying it marks a Stuck hand-off match that."
  - "internal/haltnote/haltnote.go parseRow: a history row's producer, outcome or `At` text holding ` / ` or a literal `routed to ` prefix. Settle whether the four-field split round-trips with Render and with the older three-field rows, and whether internal/reportengine, the other Parse consumer, still reads both shapes."
  - "internal/burlerengine/verdict.go quotingHint: the colon hint keys on the substring `mapping values are not allowed` in the parser's error. Settle whether that text is stable across the pinned yaml library and fires only for the colon case the hint describes, and whether the `malformedQuotedSummary` branch still takes precedence correctly."
  - "contracts/stencils/burler/burler-review-format.md: settle whether the new quoting rules are correct YAML (the indicator list, the escape rule, the example lines) and whether the diff drops or weakens any rule the old `summary` bullet carried."
  - "Scope against the issue entries #151 and #152 in the decision record: the diff has no change that re-prompts the reviewer on invalid frontmatter. Settle from the code at the review base whether the gate's existing re-prompt path already covers that bullet, and whether the claudeengine fix in internal/shuttleengine/claudeengine/session.go is licensed as the reason that path did not fire."
  - "internal/shuttleengine/claudeengine/session.go isTopBoxRule: the new `… ─` suffix test applies to any line. Settle whether a draft or output line ending in `… ─` can now be taken as a top rule and break inputBoxInterior, and whether the added test cases cover the boundary."
---

Seed focus only. The record's sections after `## Goal` are empty, so the board entry under `## Goal` is the measuring stick.
