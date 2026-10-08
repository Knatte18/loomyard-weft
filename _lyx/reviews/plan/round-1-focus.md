---
round: 1
focus:
  - "Fidelity: does some card carry each of D1 to D11 and each of the four 'Added after Discussion' decisions in the decision record, in particular the held trailing stamp (the incremental reader's `consumed` offset), the per-reader transcript-fallback turn end, and the narrowed output-files-first rule? Does any card introduce scope the record lists under Out?"
  - "Fidelity: are the doc obligations (`doc.go` of `shuttleengine`, `claudeengine` and `shuttlecli`, `PATTERN-no-denied-recovery`) and the Constraints with test-tier consequences (integration for the real-shell hook round trip, `llm` for payload capture, untagged reducer and parse tests with an injected clock) each placed in a card that touches the code they document?"
  - "Card order: which cards use a symbol or file another card creates (the reducer, prober and reader feeding the CLI verb and the three shadow-log cards; the stamp-aware activity reading before the new hooks), and does every such card come after the one it uses?"
  - "Test economy: do the reducer, reader, shadow-log and corpus cards plan new test functions for behavior an existing test or the reducer's public-surface table already covers? Name the covering test in any finding."
  - "Attack-surface question: for each new verb, flag, escape hatch or removed/downgraded guard (the `python` deny and its `claude_deny_python` switch, the `held` mark versus batten's `waitLive`, payload-reported shells waiting to Timeout, Done-at-once with leftover shells, the new read-only `lyx shuttle state` verb), does the introducing card's `**Intent:**` state what it lets through and what bounds it?"
  - "Verify coverage: does `## verify:` run the tests of every package a card targets (including `cmd/lyx`, `battencli`, `battenshed`, `orchengine`, `shedadapters`, `websterengine`, `webster cli`), and is the section in the stencil's order?"
  - "Granularity: are any cards one-per-symbol where a private type or helper belongs in its surface's card, or is any card bundling independently reviewable units (for example the three shadow-log cards, the reducer and its history)?"
  - "`ImpactSummary` on `Edit` and `Delete` cards: does each carry a real blast-radius conclusion rather than a restatement of `Intent`?"
---

First seed: the plan is large (18 cards), so the focus points at the cross-card properties a per-card read misses: decision coverage, ordering across the new-capability chain, test duplication, and the bounds on each new permission or wait behavior.
No entry restates a gate check or a `Do not flag` item.
