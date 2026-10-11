---
round: 1
focus:
  - "Fidelity, docs constraint: the decision record's Constraints require `loomengine` and `shuttlecli` doc.go updates; check whether any card (2, 6, 7, 8) targets either doc.go, or whether only `docs/overview.md` is carried."
  - "Fidelity, D5 and D9 numbers: check the cards carry the 250 s default cap, the 300000 ms Bash timeout, the 15-in-a-row give-up and the 6-call budget exactly as the decision record states them, and that no card adds scope the record does not license."
  - "Fidelity, D4 and D7: check card 4's procedure and card 9's removal point cover the relaunch and attach cases (marker kept, no new forks) and that the write-scope paragraph covers both the lead-in and the do-not-repair rule."
  - "Test economy: card 5 plans three top-level tests, card 8 a new `TestDiscussionSpec_Twill` beside `TestDiscussionSpec`, card 9 a new `TestRemoveDiscussionReadyMarker` beside the wiring test; for each, settle whether an existing table or test already covers the behavior and name it."
  - "Card order and Uses: card 6 edits `DiscussionSpec` and card 8 edits it again; check the second edit does not depend on a symbol a later card creates, and that card 2's `Uses` of `burlerCLI.awaitReviewCmd` sits after card 1's edit."
  - "Attack-surface: cards 2, 4 and 8 add a verb, fork authority and a widened write scope; check each `**Intent:**` states what the change can skip or let through and what bounds it, including the `claude_deny_agent_tool` false case."
  - "Verify coverage and order: check the `## verify:` section against the plan template's stated section order, and that every package a card targets (`internal/shuttleengine`, `internal/shuttlecli`, `internal/burlercli`, `internal/burlerengine`, `contracts/stencils`, `internal/twilldirective`, `internal/loomengine`, `internal/loomcli`, `internal/shedrecipe`) has its tests run."
  - "Member glyphs and re-sign arrows: check each `Edit` card whose `Intent` changes a signature (card 6 `composePrompt`, card 9 `Env`, `discussionWriteEntry`, `wire`) carries the arrow where parameters, results or receiver change, and none where a struct field or interface method is edited."
---

Each entry names a place and a question.
None restates a check this round's gate already enforces, and none asks for a `DependsOn` or `Produces` field or an `ImpactSummary` on a card type that does not require one.
`support-log.md` was not read.
