---
round: 4
focus:
  - "For every card that changes a constructor, required reader, deps struct or skip rule, grep the repo's test files for each caller of the changed symbol and check that every shared fixture and sibling test that breaks under the new behavior is in that card's Edit group; settle whether any fixture still builds a zero `gateslot.Config` or a hub with no `gate.yaml` where the new reader refuses it."
  - "Trace how the gate tier lists (landing, publish, round) reach `loomshed.checkedPublishFailure`, `loomshed.PublishFailureNote` and `landingshed`'s `verifyGate.recordFailure`: settle whether each function that maps an old record or fallback tier has a named source for the list in the plan's cards, and that every caller of a re-signed function is in its card's Edit group."
  - "For every test that runs the built `lyx gate test` in a scratch repo, settle whether the card's Tests text states that the seeded `gate.yaml` is committed and that the run-record directory under the worktree's `.lyx` is excluded, using `lyxdirs.DotLyxDirName` for the name."
  - "Check each warning or refusal text a card prescribes that names a `lyx config gate` step: settle whether the step named can remove the entry in every layer where the entry may sit."
---

Focus targets the method gap of unlisted fixtures and callers and the tier-list seam; each entry is a question to settle against the current cards.
