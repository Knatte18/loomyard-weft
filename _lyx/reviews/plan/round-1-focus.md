---
round: 1
focus:
  - "Verify coverage: settle whether any package a card targets has hermetic build-tagged tests (tmux, or any tag besides integration) that the `## verify:` section only vets and never runs, starting with `internal/creelcli`, `internal/creelengine` and `internal/shuttleengine`."
  - "Fidelity to the answer key: walk each Decision D1 to D9 and each Constraint in `_lyx/discussion/decision-record.md` against the cards and settle that every one is carried, in particular D6's pending-confirm rules, D3's told-once-per-change rule and D5's one `carried` entry per carry."
  - "Card order and `**Uses:**`: settle that no card depends on a symbol a later card creates or edits, in particular card 11's use of `Runner.AcceptStartupGate` from card 10 and card 8's use of the `IdleProbe` field from card 7."
  - "Attack-surface question: settle that card 4's early end of the connect wait, card 10's `Runner.AcceptStartupGate` and card 6's shorter two-miss guard each state in `**Intent:**` what they can now skip or let through and what bounds it."
  - "Reachability of a prescribed refusal: for each refusal or guard branch a card prescribes (card 2's stderr fallback, card 4's non-race exit, card 10's dead-strand and unreadable-record returns), settle that a named input and a named test reach it and that no upstream check rejects that input."
  - "Edit-card glyph shape: for each `Edit` card whose `Intent` changes a parameter, result or receiver, settle that the member glyph carries the re-sign arrow, and that no `Edit` card names specific symbols while listing only non-test Go file paths."
  - "Test economy: settle that each new test row or function names no behavior an existing test already covers, and that no card asserts a stencil's static prose."
---

Seeded from the plan, the decision record and `lens-review.md` of the plan directory.
Mechanical checks of the validation-checks section are left to the gate.
