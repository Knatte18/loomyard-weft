# webster: halt note truncates drift gate message; no way to accept expected drift

`webster-drift-halt-truncated` · Note · bug, webster, drift

Imported from [issue #172](https://github.com/Knatte18/loomyard/issues/172).

## What happened

A run halted `blocked` at Webster (anomaly `escalation-to-human`) after the plan changed and was rebaselined on entry, keeping 1 begun batch.
The verify gate then failed: "verify gate did not pass after 3 of 3 re-prompt(s): the run-level "drift" entry failed: unexpected cell: changed in case ...".
The drift findings were numeric cells in a captured baseline file that moved in the sixth significant digit (for example an integer value changing in its last digits and a rate changing in its last decimal).
The halt note's message is cut off mid-finding, in the middle of a quoted value, and the way-forward text ("1) fix what the "drift" entry's findings name and commit it on the task branch; 2) re-step the Webster row") is cut off after step 2.

## Why it is abnormal

1. The halt note truncates the gate message, so the full list of drifted cells and the complete way forward are not in the note.
2. Drift in the last digits of a captured numeric baseline is the shape of a legitimate behaviour change from the card's own work.
   Three re-prompts of the implementing agent cannot "fix" it, and the way forward offers no route to accept it as expected.
   The gate looped to its re-prompt limit on a finding that needs an operator or parent decision.

## What to do

- Stop truncating the halt note's message, or write the full gate report to a file under the friction directory and name it in the note.
- Offer an explicit way to accept an expected drift (for example a re-baseline verb for the "drift" entry), and say so in the way forward.
- Consider ending the re-prompt loop early when the same drift findings come back unchanged after a re-prompt.

Provenance: task `wemod-invariants`; note `loom-halt-2.md`.
