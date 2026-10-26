---
round: 3
focus:
  - "internal/websterengine/runlevel.go `Run`: does every refusal it adds for an unreadable decision record sit on a path a test reaches, given that `discussionparser.DeclaredPatternChanges` and `discussionparser.AddedDecisionTexts` both read the same record file and return the raw read error, and does the refusal name its way forward per PATTERN-refusal-way-forward?"
  - "internal/loomcli/drivewiring.go `goDriverDepsFor`: does a test fail if the `arbiterSeam.JudgesFirst` assignment is removed or aimed at another row, so that only the Webster row is marked, and is the predicate pinned once rather than twice?"
  - "internal/websterengine/template_test.go master `parent_decisions` rows: do they assert only what Go fills into `parent_decisions` (the entries in order, or `none`), with no assertion on a stencil's static prose or section headings?"
---

Focus targets the sites round 2 reported, stated without reference to prior findings.
