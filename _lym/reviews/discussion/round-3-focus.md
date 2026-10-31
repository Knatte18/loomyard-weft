---
round: 3
focus:
  - "D4 Record-Write's board-body fence: check that the fence length is defined for every body, including one with no backticks and one with inline code only, so CommonMark opens it and `fencedLines` in internal/discussionparser/append.go treats the body as fenced; settle whether `decision add`, `locateOpenRisks` and `CheckDecisionIDs` then read only the record's own headings."
  - "D4 and D5 readers of the record: check that every function reading required headings, including `missingSections` in internal/discussionparser/validate.go, ignores fenced lines by one rule, so the Darn-Write validate-only guard cannot pass a record whose real `## Open risks` heading was deleted."
  - "D4 Record-Write crash behaviour: settle whether a record or support log left truncated by a crash is repaired or ignored on resume, given that Darn-Write's `discussion` handoff gate re-prompts a writer that D6 forbids to edit the record."
  - "D13 retired-key refusal: settle at which point of `PrepareRunSettings` in internal/configreg/runsettings.go the check runs relative to `moveRunSettings` and `module.Migrate`, so Loom's `MigrateConfig` drop list cannot strip a retired key before the check; and settle what happens to a hub base `loom.yaml` or `darn.yaml` value of a retired key."
  - "D6 and D7 with no parent: settle what the writer does when a `Decision for <slug>:` note answers its stop and no `Added after Discussion` entry exists, given the record is the only key the perch reviews against."
  - "D4 in-flight darn runs: check each claimed route (goto Record-Write then resume, and a resume without the goto) against what internal/driftengine does when a scope record or passed capture already exists and when the decision record is missing."
  - "D4 `decision add` on a darn run: check that every refusal in internal/loomcli/decision.go that a darn run can reach names a way forward that exists after Record-Write replaces the Darn-Write session as the record's writer."
---

Round 3 reviews the fixer's changes to the record fresh; the entries above name where to look and which question to settle.
