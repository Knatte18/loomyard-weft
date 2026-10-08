---
round: 2
focus:
  - "decision-record.md D3 (reducer rules) and D4: for every input a state rule reads beyond the hook signals (provider-process liveness, the output-file list, run mode, any transcript marker), does the Decision name its source, and does every state have a defined `since` stamp, including states caused by a transcript marker or a missing output file?"
  - "decision-record.md D11 (python deny) with Open risks: does the Decision itself state what the command-position match lets through and what it falsely denies, that the deny is a guardrail rather than a barrier, and when the per-repo switch is read and by whom?"
  - "decision-record.md D8 (background shell wait) with D7: for a payload-reported shell that never finishes, and for a fork and a shell outstanding together, is it stated what signal reaches the parent or operator, and what the outcome is at Timeout?"
  - "decision-record.md D1 (new hooks and the `lyx_at` stamp): does the Decision state what a hook command may do to the prompt (output, blocking) and how far the agent-writable stamp is trusted, per the attack-surface question?"
  - "decision-record.md D3, D5 and D8 together: where the wait loop acts on a turn end that D3 classifies differently, does the text say which side is authoritative, and does the D5 disagreement-log criterion agree with that?"
  - "support-log.md Rejected alternatives against decision-record.md D3: does the stated reason for rejecting a persisted state file hold for states derived from call-time facts?"
---

Entries restate their sites; each asks for a settled answer in the Decision text the plan writer reads, not in `support-log.md`.
