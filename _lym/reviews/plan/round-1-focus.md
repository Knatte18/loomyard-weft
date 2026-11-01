---
round: 1
focus:
  - "Cards 10, 11, 12, 13: does each decision-record bound (D1 claim and lock waits, D3 lease and abandoned mark, D6 killed-runner line group) have a card that carries it, and does each new refusal or guard branch (runner refusals, start failure, marked exit file, recorded 75 replayed as 1) name an input and a test that reaches it, with no upstream check rejecting that input?"
  - "Cards 10, 11, 13: each new verb, escape hatch or removed guard (hidden `run-detached` verb, capped-vs-in-process choice by strand, `testing.Testing()` launch refusal) states in its Intent what it can skip or let through and what bounds it."
  - "Cards 10 to 13: does any test row plan a new test for behavior an existing test already covers, or one test per symbol, where a public-surface test covers it?"
  - "Card order: does any card use something a later card creates, in particular cards 6 and 7 against 13, card 8 against 10, card 9 against 12, and cards 3 and 2 against 4, 11, 12 and 13?"
  - "Edit cards with source-file targets and Intents naming specific symbols (11 and 14 especially): are member glyphs listed, and do members whose parameters, results or receiver change carry the re-sign arrow?"
  - "Cards 1, 2, 3, 8, 9, 10, 14: is each `Extends` ref only added to, never rewritten, reordered or trimmed?"
  - "Decision-record fidelity: is each Decision D1 to D9 and each Constraint carried by some card (D7 render beside `GateCall`, D9 shellguard row and stencils, the `battencli/land.go` strand removal in card 5), and does any card add scope the record does not license, such as `cmd/testtiming/resources.go`?"
  - "ImpactSummary on Edit and Delete cards (11 and its Delete group in particular): does it state a real blast-radius conclusion rather than restate Intent?"
  - "Verify lines in all cards: any raw test command instead of one `lym gate test` call with a tier list and scope form?"
  - "PATTERN fit: cards against the entries named in Shared Decisions and the `PATTERN changes` section (PATTERN-keep-warm-cap, PATTERN-shuttle-provider-seam, PATTERN-test-speed, PATTERN-gate-slot, PATTERN-tolerant-records)."
  - "Research steps: does any card leave a fact to be found while it runs, or carry a rule to halt on a discovery?"
---

The plan has 15 cards in a layered order: config and resolver, strand-free verify commands, run-record changes, then the runner and front.
Cards 10 to 13 are by far the largest, so most of the focus goes to their guards, bounds and test rows.
`support-log.md` and the `support-log.lens-*.md` files were not read.
