---
round: 1
focus:
  - "Fidelity: walk every Decision and Constraint in `_lyx/discussion/decision-record.md` (D1–D8 and the Constraints) and settle which card carries each; also settle whether any card's scope, notably card 7's `gateQuiescenceFailureMessage`, `gatedRowFanViolations` and `TestNoGatedRowAuthorizesForkSubagents` edits, is licensed by the record."
  - "Card order across cards 5, 6 and 7: card 6 uses `recordRoundFans` and `readRecordedFans`, and card 7 creates `plan:internal/shedadapters#RoundFans`; settle whether each symbol a card uses is created by an earlier card, not a later one, and whether `Uses` lines name symbols that exist at that point."
  - "Reachability of prescribed refusals in cards 3 (`twillEngineRefusal`, fork-audit violations, single-fixer start refusal), 5 (refusals on a round with no accepted review, unrecorded fans) and 8 (`fanShapeError`, retired `fan_review`): for each, settle the named input and the named test that reaches it, and whether an upstream check rejects that input first."
  - "Attack surface: card 3 adds a forking fixer and card 7 changes the gate-quiescence guard's premise; settle whether each `**Intent:**` states what the change can now skip or let through and what bounds it."
  - "Test economy: for the new and edited tests in cards 1, 2, 3, 5 and 6, settle whether an existing test already covers the behavior, and whether any card plans an assertion on a stencil's or lens's static prose against the Shared Decisions rule."
  - "Edit-card glyphs: for cards whose `Edit` targets include non-test Go files, settle whether `Intent` names symbols that need member glyphs, and whether any member whose parameters, results or receiver change lacks the re-sign arrow (cards 3, 5, 7, 8)."
  - "Verify coverage: settle whether every package any card targets, including `contracts/specs`, `internal/configreg`, `internal/configcli` and the testkit packages, is run by both `verify:` commands, and whether the section form is admissible."
---

Focus set from a read of `00-overview.md` and the card file headers; no prior round exists.
