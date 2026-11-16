---
round: 3
focus:
  - "Crash re-entry order (D12, D6): for an attempt recorded in flight, does the decision record attach to the attempt's strand under its gates before it records any report found on disk, so a strand still re-prompted on a failing gate is never recorded done early? Settle whether a next batch can spawn beside a live strand."
  - "Strand lifecycle (D6, D13, D12): does every way an attempt can end without being recorded done, including a failure, a timeout, a machinery loss, a NEEDS_PARENT stop and a fault in Weaver's own state, leave its strand stopped before the next attempt or the arbiter starts, and does the order of recording the failure against stopping the strand survive a crash between them?"
  - "Batch identity for the shared checks (D3, D11): does the record fix the key a batch is stored under and the batch list the shared checks receive, so a pending batch with no record reads as unbegun, and only the run's last batch reads as last?"
  - "Reset and re-entry bounds (D1, D2, D7): for each guard the record adds or relaxes, such as the reset, the move-aside of a corrupt state file and the no-spawn path, does the Decision state what it can skip and what bounds it? Check the reset against a strand that outlived a crashed producer, and the plan and records that live in the weft worktree rather than on the task branch."
  - "Verify fixer (D10, D12): does a verify fixer strand have a fixed output-file identity recorded before its spawn, so a crash during it attaches rather than spawning a second fixer?"
---

Round 2 left the D12 crash guard, the strand lifecycle and the batch-identity questions open; the focus items above restate them without a verdict.
Two `decision` findings for the parent, on who pushes the journal mid-row and on reading "each batch is a producer", are not for the reviewer to re-raise unless the record's Open risks no longer carry them.
