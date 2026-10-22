---
round: 1
focus:
  - "D7 and D8, decision-record.md Decisions: the claim-then-interrupt path lets an interrupt through a pause. Settle whether each state of the claim (entry gone before the key, claim succeeds but the key fails, a strand that is not busy) is bounded by the Decision itself."
  - "D1, D3 and D8, decision-record.md: each fails open on an unreadable record or a name that does not parse. Settle whether the Decision states what that lets through and what bounds it."
  - "D4, decision-record.md: a verify command is stopped by SIGTERM then SIGKILL on its process group, and restarted after the pause. Settle whether what the stop can reach and what bounds it are stated, and whether any fact it rests on is left for Plan or an implementer to find out."
  - "D2 and D6, decision-record.md: callers are told to hold, retry, claim or refuse. Settle whether each caller listed has one stated outcome and whether any caller of the chokepoint is left to be found later."
  - "Acceptance criteria against Decisions: settle whether each Decision's bound has a criterion that reaches it, and whether Constraints or Open risks carry any deferred research question."
  - "support-log.md against decision-record.md: settle whether any requirement that the plan writer needs lives only in the support log's lens-review findings (Plan-Write never reads it) and so must be extracted into the record."
---

Seeded from a read of both files and the discussion spec.
The rubric's Do-not-flag items (optional notes subsection, rejected alternatives in the record, call-site enumeration) are left out of every entry.
