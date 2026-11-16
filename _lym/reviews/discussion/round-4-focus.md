---
round: 4
focus:
  - "Verify fixer and the gate-fixer pattern: in decision-record.md D10, D12 and the PATTERN-gate-slot constraint line, does the verify fixer run through gatefixer's spawn-or-attach and await with a Weaver-owned store, or through a direct shuttle spawn the pattern admits? Settle whether the crash window between spawn and record save is bounded, and whether a verify runner crash gets no fixer."
  - "Lym weaver reset live-strand check: in D2 and D12, does the refusal cover every attempt and fixer strand whatever its recorded outcome, including a strand whose guid was never recorded, found by its output files? Settle whether the check runs before any state is archived or cleared."
  - "Attach error and failed stop: in D6 and D12, state what the row returns when shuttle's attach returns an error, and when the stop of a finished attempt's strand fails, and whether any next attempt or arbiter can start beside a live strand."
  - "Corrupt-state way forward in D1 and D9: is restoring an earlier committed state version bounded so it cannot lose the batch's begin record or an in-flight strand? Settle what the batch-start commit becomes after a restore."
  - "Reset archive in D2: does the record name where the reset archives Weaver's state, and whether that location is read by D13's prior-generation readers and committed with the plan restore?"
---

Focus targets the findings still open after round 3.
