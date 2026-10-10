---
round: 1
focus:
  - "decision-record.md Scope against Decisions: does Scope name every package a Decision changes? Check D7's agent deny of the new variable, D10's four runner packages, D12's `gitoracle` parity rows and D8's reuse of the `reedcli` watchdog recognition; a Decision that changes a package Scope omits is a consistency finding."
  - "D7, D8, D9, D12 and D13 as guards or escape hatches: each states a bound, but does the bound say what the mechanism can skip or let through? Check D7's forged-variable bound, D8's watchdog kill without failure and its escape by argv, D12's unhonoured `GIT_DIR` and `GIT_WORK_TREE`, and D13's call-site-only scan."
  - "D6 and D4 together: D6's runtime check relies on tmuxkit-owned socket directories and `Main` being called. Does a tagged package that never calls `Main` or a tier-1 or tier-2 package bypass the check unbounded? Settle whether D6's bound covers it."
  - "D11 `CopyHub`, `SharedHub` and shape set: is the relocation and write-check contract complete enough for the plan writer to classify tests without reading the support log? Check that the 'write' definition and the closed shape set are in the Decision or Constraints, and that nothing the plan writer needs lives only in `support-log.md`."
  - "Goal and Acceptance criteria against Decisions: is every Goal bullet (including 'a package fails when a test leaves a child running') covered by a Decision and a criterion, and does any criterion test something no Decision states?"
  - "D12 rationale and D1 figures: the 38% `rev-parse` share is cited without a source in the record. Settle whether it is a premise a decision rests on or only motivation, and whether D12 is sound if the figure is wrong."
  - "Open risks and Auto-mode assumptions against Decisions: does any assumption (permanent watchdog allowance, `exit-empty off`, `$SHELL` non-login) contradict a Decision's stated bound, or is any risk's named mitigation absent from the Decisions?"
---

Seed focus only; every rubric rule, including the `Do not flag` list, still applies in full, and no entry caps severity or pre-states a verdict.
Entries avoid asking for signatures, call-site lists or rejected alternatives in `decision-record.md`, which the writer stencil told the discussion writer not to gather.
