---
round: 1
focus:
  - "support-log.md Question ledger Q8 says the inject TTL is wall time, but D9 and D14 in decision-record.md count it only while the receiver is live and unpaused: settle which the record means and whether the ledger entry is stale."
  - "D9 makes an orch strand a privileged injector while D16 refuses any inject into an orch: settle whether the privilege rule and the orch exclusion leave a path where an orch's inject reaches another orch, and whether each Decision states what bounds it."
  - "D4 sender identity: settle whether the Linux ancestry walk bounds the privilege escalation it claims to close, including a peer in no strand pane, a program sender claiming an upper-case name, and a $LYX_STRAND_NAME that disagrees with the ancestry."
  - "D10, D13 and D17 each add or widen an edge (guarded interrupt, renewable hold, record-less send into any live reed strand): settle whether each states what it can skip or let through and what bounds it, per the attack-surface question."
  - "D6, D12, D15 and D19 on crash and restart: settle whether the journal fold, carry-forward, reconnecting waiters and the removed-holder sweep leave any state a restart could lose or duplicate, judged against the Acceptance criteria that claim to cover it."
  - "Check each Acceptance criterion traces to a Decision and each Decision's observable behavior (D7 backoff, D8 ack, D11 forward, D18 log) has a criterion where a plan writer would otherwise have to guess."
  - "Check whether deliberation narrative sits in decision-record.md that belongs in support-log.md (D3 and D7 board-departure rationale, Auto-mode assumptions repeating Decisions), extracting any requirement into Decisions or Constraints before proposing a move."
---

The first five entries target the highest-risk decisions: privilege, identity, bounded edges and persistence.
The last two cover traceability and relocation, which the rubric treats as equal in standing to gap-filling.
No entry flags a missing Notes subsection, missing rejected alternatives in the record, or incomplete call-site enumeration.
