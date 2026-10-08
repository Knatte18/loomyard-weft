# Repair 1: Discussion-Burler, history_length 6

Failure: `producer` error, not transient.
The `shuttle: Send` to the burler round 2 attempt 2 left the text pending in the input box after 2 extra Enters, so the skill load did not land.

Trace lines acted on (`trace-20261008T062533Z-170f1ceca72e57c5-207416.log`):

- `burler: stopping a live half of a round strandGUID=6e8ab264ee0d06be85628411512709bb`
- `shed: step refused kind=producer transient="" error="... Send: the sent text is still pending in the input box after 2 extra Enter(s); the submission did not land"`

The only `fabric: mutation` records are a records commit and its push; there is no half-finished add or remove, so there is nothing to restore.

Action: re-step (`interrupt_policy: reinvoke`), the state `failed` being resumable.
Outcome: see the stop report.
