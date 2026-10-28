---
round: 1
focus:
  - "internal/landingshed Finalize `mergeParent` and `mergeIntoTask`: does a conflict or overlapping-file merge now always run the conflict session under the parent lock and go on to the `finalize` gate for every recipe, and does the retry after a merge-in-required error merge, gate and land instead of rerouting?"
  - "Removed seams (`Deps.ConflictReroutes`, `Deps.LargeMergeLands`, `taskMerger`, `WriteOverlapNote`, `OverlapNotePath`, `overlapNoteInput`, darn's `ReviewInput` override): does any reference, doc comment, recipe comment or test fixture to them remain, and is the merge record's `Overlap` still written for the fixer?"
  - "`fixerOutcome.haltReason` and the new escalate prefix: is the prefix distinct from `LandingStopPrefix` and `shedadapters.DecisionHaltPrefix`, is the predicate the prefix's only reader, does `landingshed.HandsBack` stay false for it, and do `EscalateJudgedFirst` true/false on loom/darn vs pawl select the right prefix and the right way-forward text (no `lym loom decision add` on pawl)?"
  - "`sheddriver.ArbiterSeam.JudgesFirst(producer, reason)`: is the reason read from Reason for a stop and from Error for a status-built stop, does only Webster or the escalate predicate mark a stop, and does every other blocked Parent-Align or Finalize stop still meet the tackler first?"
  - "`restep_hold` on the loop envelope and `lym shed status`: is it computed beside `hand_back` from a second told check, false when no check is told, and do the driver stencil and guide skip re-launch on `binary_changed` and the binary watch at a stop marked `restep_hold` without the stencil reading reason text or naming a recipe?"
  - "Batten's binary re-step `HandBack` seam in the batten CLI wiring: does it skip a stop the escalate predicate matches as it skips a landing halt, and does a pawl or non-escalate stop behave as before?"
  - "Every other landing halt (spent cap, over-cap change, unusable turn, undecodable record, lock re-check, reroute-cap) keeps `LandingStopPrefix` and its hand-back; does the diff change the Finalize `on_reroute` edges of loom or darn beyond comments?"
  - "Docs per D5 and the declared `PATTERN-shed-stop-chain` change: do the orch stencil, recipe comments, Code-Review rubric, fixer report spec, `docs/overview.md`, loom status spec, driver stencil and guide, and the `doc.go` files (`landingshed`, `arbiterstop`, `sheddriver`, `battenshed`) carry the new behaviour, and does `PATTERN.md` carry the declared entry text?"
  - "Test economy: do the new tests in `landingshed`, `loomcli`, `sheddriver`, `shedverbs` and `battenshed` cover each added behavior once, without duplicating an existing test or asserting stencil prose?"
---

Seeded from the decision record's D1 to D5, its acceptance criteria and its PATTERN changes.
No finding from a prior round exists to carry.
