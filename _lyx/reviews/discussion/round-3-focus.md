---
round: 3
focus:
  - "Check that the rewind-picker decision (detect an open picker, close it with one Esc, re-check the input box, report a failure) is stated once, as an authoritative Decision, and that Scope, D6, D7's PATTERN rule, the Acceptance criteria and `support-log.md` (Rejected alternatives, Interview, Q4) no longer say the opposite."
  - "For that picker decision, settle whether the Decision states the detection capability and where it lives under `PATTERN-shuttle-provider-seam`, which code path sends the closing Esc, its bound (at most one Esc per detection, never selects an entry, the re-check-fails case), whether resume then sends its line, and which surfaces (pause, resume, shuttle send, shuttle interrupt) run the check, each with an Acceptance criterion."
  - "Check that D7's Inject refusal covers every input list containing the provider's interrupt key (a lone Escape, two Escapes, Escape with a SettleMS, Escape among other keys), and that the stated bound matches what the refusal enforces."
  - "Check that resume's pre-send check agrees with D5's `no_running_turn`: a session whose raw state reads busy but whose turn-start hold released must get the resume line, not a refusal, and D1's list of raw-state predicate uses matches."
  - "Check that D3's Bound states what the running-turn needle matches in a capture, including a transcript line quoting the hint, and what limits the Escs that false positive lets through."
---

Focus targets the round 2 BLOCKING and MEDIUM areas; no entry caps severity.
