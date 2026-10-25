---
round: 1
focus:
  - "D7 and D6: `lyx drift accept` under scope `partial` has no count or content bound, and `Drift-Verify` is Stuck on an accepted drift only under `full`; settle whether the Decision states what an acceptance can let through under `partial` and what bounds it."
  - "D1 and D4: the `capture` and `compare` commands, the `--skip` list and the `drift.enabled` switch can each skip or weaken regression; settle whether each Decision that introduces one states what it lets through and what bounds it, including the flag turned off mid-run."
  - "Open risks, second bullet: the Models-side facts (`NORCE_BASELINE_DIR` in verify mode, the capture's exit code, a compare that discovers no test) are left to be found out later; settle whether the design rests on any of them, and whether the Out scope and the writer's instructions make them a fact the writer was told not to gather."
  - "Cross-check D2, D3, D5 and D6 against the Acceptance criteria and each other: the `Drift-Await` outcomes with the flag off, the re-entry rules after a dead capture, the halt row that `lyx drift accept` and `lyx loom goto` name, and the darn variants. Settle whether any two statements contradict."
  - "Relocation: settle whether D1's, D4's and D5's `Rationale` paragraphs, the D4 clock and Windows detail, and the Constraints bullets that restate a Decision carry a requirement the plan writer needs; extract what does, and name the rest as belonging in `support-log.md`."
  - "Decisions that touch the board or file work: settle whether any Decision, Open risk or Note changes this task's own board entry, or files or promotes a follow-up as a task, including the OSL-1046 follow-on and the parent-position question."
---

Seed focus from one read of both artifacts.
Each entry names a place and a question; none states a verdict.
The `Do not flag` items (the optional notes subsection, rejected alternatives in the record, call-site enumeration) are not promoted.
