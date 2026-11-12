---
round: 3
focus:
  - "In decision-record.md D6 and D8, settle whether an operator message in a bridge channel that sits after a held peer message is handled exactly once across repeated bridge polls: a held message keeps the channel's delivery cursor still, so check what stops an operator approve, a later revoke and a plain-text line from being read and handled again, and whether the revoke stands."
  - "In decision-record.md D4 and D8, settle what an adopted or resync-added bridge channel reads and handles when it enters the bridge map: envelopes older than creel's journal retention and operator commands already in the channel must not be redelivered or re-handled."
  - "In decision-record.md D3, D7 and D8, settle the lifecycle of a held machine-outbound message: whether the relay can post a message creel already returned at its hour, where the refusal count is kept, and how a machine-outbound event is journaled so the retention carry and a binary that does not know it read it safely."
  - "In decision-record.md D4 and D8, settle the outcome for a hand-archived bridge channel found by name_taken, and where the unpinned-creator first-contact notice goes when the prime orch has no open channel."
---

Focus targets the bridge channel's read and post lifecycle, where the settled design still leaves paths open.
