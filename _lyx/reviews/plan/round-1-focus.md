---
round: 1
focus:
  - "Fidelity, decision record D3 and D7: find the cards that carry the fail-open Warn on an activity-query error, the `State.Stuck` marker for a held idle-phase notice (left alone by the passing idle probe, cleared only by a passing send), and the rule that a held step never counts toward `maxConsecutiveTickErrors`. Settle whether each has a card whose Intent states it."
  - "Fidelity, D5: find the cards that replace the 48-character needle in `boxHoldsSentText` and the prefix acceptance in `isOwnText` with the exact match, bound the placeholder acceptance (collapse threshold, line count, per-Enter activity check), and make `clearUnlandedText` clear only an exact-match box. Settle whether any of these bounds is missing from the card that introduces the guarded Enter."
  - "Fidelity, scope: check each card against the decision record's In and Out lists. Settle whether any card touches `Runner.Interrupt`, `Runner.Inject`, `InterruptSequence`, `ModelSwitchSequence`, `awaitStartup`, or a queue module, or licenses scope from the carry-over block."
  - "Attack-surface question: for the activity-query fail-open (card 2 or 5), the guarded Enter and placeholder acceptance (card 6), the removal of the leading Escape (card 9), the retry with no deadline (card 14) and the `driver_busy` refusal (card 13), settle whether the introducing card's Intent states what can now be skipped or let through and what bounds it."
  - "Card order: settle whether any card's Uses or plan-prefixed symbols are created by a later card, especially cards 4 to 9 (the `ReedOps` seam, `TextPaster`, `pasteVerified`, `PasteSubmits`) and cards 12 to 15 (`StartDriverBusyKind`, `ErrChildDriverBusy`)."
  - "Test economy: settle whether the new rows and test files in cards 1, 2, 7, 13 and 16 duplicate behavior an existing test or a sibling card's public-surface test already covers, naming the covering test."
  - "Edit cards 5, 6, 8, 9 and 11: settle whether every member glyph whose parameters, results or receiver the Intent changes carries the re-sign arrow, and whether any Edit card names symbols in its Intent while targeting only non-test Go file paths."
  - "Verify coverage: settle whether `## verify:` runs the tests of every package a card targets (including `internal/shuttlecli`, `internal/loomcli`, `internal/battencli`, `internal/battenshed`, `internal/orchengine`, `internal/reedengine`, `contracts` and `cmd/lyx`), and whether the section follows the plan template's order."
---

Focus is where to look, not a verdict.
The `Do not flag` list was checked: no entry asks for a dependency-edge field, an `ImpactSummary` on a non-`Edit`/`Delete` card, or a gate-enforced check.
