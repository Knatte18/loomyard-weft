---
round: 1
focus:
  - "internal/shedadapters/parentseat.go: does the asking rule match D2 (no ask yet, seed differs, Discussion-Write history after the latest ask, done-with-findings from the round before, not-sent from an earlier round) and is an ask already in the round posted whatever the key reads?"
  - "internal/shedadapters/parentseat.go and internal/burlerengine/parentreview.go: ask record state machine per D3 and D5 (pending-post re-post to the stored strand, open keeps the first deadline, no-parent on no strand or post error, timeout closed under the per-run lock before acceptance, not-sent when the chair timeout leaves no room); is the seed compared at full nanosecond precision?"
  - "internal/orchcli/notify.go: the new post variant takes the resolved strand and signs as the program LOOM alone, ignoring a non-orch strand claim; are its failure paths (no receiver, post error) handled as D3 says?"
  - "internal/burlerengine parent coverage gate entry: listed after the review-parse entry, present only for a round holding an ask for its seed and round, never passes on its cap, fails naming every uncovered report id; check Finding.ParentIDs parsing in verdict.go."
  - "internal/burlerengine parentseat.go and the copy beside the review: done report copied once the review is accepted, copied on a restart after acceptance, and a later answer refused; parent consolidation part filled whenever the round holds an ask, not-sent included."
  - "internal/loomcli wiring: discussion_review_ask_parent reads through the setting lookup, an unreadable value logs and counts as false, and the parent seat lands on the Discussion segment only."
  - "contracts/stencils (burler-parent-consolidation.md, burler-review-format.md, burler-step-review.md) and contracts/orchrole/orch-role.md: read as the diff leaves them against the Review criteria for D6 and D7 (await verb, origin: parent, parent_ids, BLOCKING default, decision mapped to design, never rejected; orch fork exception, two verbs for the named slug only, findings file shape, D1/D2/D5 keys)."
  - "Test economy: the contained candidates TestComposeReviewPrompt_AssistRules, TestParentCopyPath, TestParentCoverageGateEntry_Shape, TestParentRulesBlock, TestComposePrompt_RendersEngineBlocksForTheHalfsEngine and TestTemplate_FillsWithAllMarkers, checking assertions against their covering tests."
  - "Scope: docs/overview.md, internal/shedrecipe changes and internal/burlerengine/assist.go edits against what the record licenses; D1 asks for no new agent-fan code."
---

Seed focus drawn from the decision record's D2 to D7 and the diff's file list.
