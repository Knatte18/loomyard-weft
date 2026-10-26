---
round: 2
exclude_lenses:
  - board-fit
focus:
  - "Beam entry run hold: in decision-record.md D9, D3 and D11, read the rule that says when a beam's pawl run holds the beam entry against landing and when it does not. Settle whether one rule over named pawl rows covers every reader of a run-held entry's status: the membership check in boardengine's project check, the run lock, the fork precondition, loom peers, and the Project layer."
  - "Parent dobby capacity: dobby counts every run it drives, halted ones included, against max_runs. Read D10 and D11 and settle whether a pawl run parked at Beam-Await for a long time can keep its parent's dobby from starting ready tasks, and whether the record states the bound."
  - "Beam-Await accept ordering: read the steps D9 runs on an accepted land request. Settle whether a member can start or fork between the land re-check and the dobby stopping, and whether a crash between any two steps leaves a state the re-entered row recovers from."
  - "Adopt and ownership: read D6 and the acceptance criteria. Settle whether adopting root before a beam, or adopting a beam whose landing is in flight on another machine, leaves a pawl run, dobby or orch with no driver or with two."
---

Board-fit ran in round 1 with no findings, so it is excluded.
