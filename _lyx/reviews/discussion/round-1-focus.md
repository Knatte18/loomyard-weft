---
round: 1
focus:
  - "D1, D3, D5, D7, D8: for each new verb, program-only edge, hold, TTL or removed guard, does the Decision state what it can now skip or let through and what bounds it? Check D1's removal of the stage 1 withholding against its stated bound."
  - "D5 restart and outcome handling: do the pending-with-no-id lookup, the `unconfirmed` advance rule and the `expired` requeue leave any step that can be typed twice or skipped, given the stated journal lookup and the never-typed-twice rule in D3?"
  - "D4 note gate: does the reply validation (sender, `reply_to`, path, non-empty file, `defer` in soft only) bind every way a cycle could reach `/clear` without a written note?"
  - "D9 against Scope Out and the Goal: is the retained `SessionIdle` and guarded interrupt consistent with 'the idle probe goes' wherever the record states it, and with `PATTERN changes` for `PATTERN-orch-pane-single-writer`?"
  - "Consistency between Decisions, Scope, Constraints and Acceptance criteria: does every Decision's observable behavior (D2 reminder, D6 sender rule, D8 cleanup) have a matching scope line and criterion, and does any Decision ask the run to edit this task's own board entry?"
  - "Relocation: does any Decision, Open risk or Auto-mode assumption carry deliberation narrative that belongs in `support-log.md`, after extracting any requirement the plan writer needs into Decisions or Constraints first?"
---

Seed focus only; each entry names a place to look and a question to settle.
