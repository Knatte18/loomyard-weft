---
round: 3
focus:
  - "decision-record.md D13 (empty-reason paused stop): settle whether the reason it relies on exists anywhere durable. `shedengine.Status` carries no reason field, a reason lives only on the step result and envelope, and `Goto` persists `StatePaused` without being a step stop. Does D13 name where a reason is set, and does D12's status re-read avoid classifying a routine goto as an empty-reason paused stop?"
  - "decision-record.md D4 (shared verify retry count) against batten's publish-retry marker, which sits in the batten prime run's scratch directory in another worktree: settle how the Go driver and batten each learn that the one retry is spent when no batten prime run exists, and whether any actor writes a file it cannot locate."
  - "decision-record.md D7 (tackler bound): settle what mechanically bounds a tackler's worktree mutations beyond stencil text. Do commits, HEAD or branch moves, and edits to files already dirty at spawn pass the post-run comparison of the stop key and uncommitted paths?"
  - "decision-record.md D7 tampered rule against D12 moved-run rule: settle which wins when an orch goto or operator edit changes the stop key while a tackler runs, and whether the cause recorded blames the tackler."
  - "decision-record.md D3 hand-over row against D5: settle whether the binary-eligible row keys on a field computed against the calling lyx's own build or on the on-disk build identity D5 mandates."
---

Check each entry against the current text of both files; the attack-surface question applies to the tackler goto and to any new marker or comparison.
