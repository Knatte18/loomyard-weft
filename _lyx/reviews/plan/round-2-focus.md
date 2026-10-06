---
round: 2
focus:
  - "Skill-loading tests in `internal/shuttleengine` (`startup_test.go`, `session_test.go`, `fakes_test.go`) and `internal/shuttleengine/claudeengine`: settle whether some Shared Decision or card states they, the `SkillLoader` fakes and the loomcli stub driver scripts are left untouched, and that a redundancy candidate among them carries `//testtiming:keep` naming `skills-in-one-turn`, and whether the prune procedure and the shuttleengine prune card reference it."
  - "Write scope of each card that edits or untags tests (the six package prune and untag cards and the final measurement card): settle whether its `Targets` cover every path its `**Intent:**` lets it change, including shared fakes under `internal/testkit/` and conditional test-file fixes."
  - "Untag procedure and the fabricengine untag card, against `PATTERN-hubforge-fixtures`: settle whether a test that needs a real hub is told to build it through `internal/hubforge`, and whether a moved test is told to drop the hub and never hand-assemble one in a temp dir."
  - "Card that adds the out-of-process exclusion scan: settle whether the `os.Args` rule matches the decision record's index-0 wording or is stated as a deliberate bound."
  - "Card that adds the keep directive: settle whether one definite planned test shows a kept test still counted among another candidate's covering tests, with no conditional wording."
---

The focus entries restate the sites to check; they cap nothing.
