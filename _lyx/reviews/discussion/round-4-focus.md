---
round: 4
focus:
  - "Failure signal of the guard invocation (decision-record.md D3, D4, D5 step 2): D3 puts `source-file <path>` at the head of D5's guard invocation, D5 step 2 chains the `destroy-unattached` and `exit-unattached` pins after it as fatal, and D4 says a non-zero `source-file` exit is a warning. Settle whether the decision tells the plan writer how a source failure is told apart from a pin failure when one tmux client exit status carries both, and whether an operator file with a typo can still fail the boot or leave the pins unapplied."
  - "Acceptance criteria for the file-error path (decision-record.md acceptance criteria 9 and 10, and the pin criteria): settle whether a criterion covers an operator file that errors at run time and also sets `exit-unattached on`, requiring the boot to proceed, warn and leave both pins `off`."
  - "Premise of atomic queues (decision-record.md D3 and D5 rationale): the text says tmux runs one client's command queue to completion before another's. Settle whether that holds when the sourced file holds a waiting command such as `run-shell` without `-b`, and whether any conclusion that depends on it, such as the claim's check-and-set or the chained spawn pins, rests on it where it fails."
  - "Starter that never reaches its guard invocation (decision-record.md D3 step 1 and step 2, and D3's Bound): D3 says the claim always goes to the server's first starter. Settle whether the decision states the outcome, and what clears it, when that starter's boot ends between its spawn and its guard invocation and the server stays up with no claim and no sourced file."
---

The first two entries concern one defect seen from the decision text and from the acceptance criteria.
Entries three and four are residual checks and do not outrank the first.
