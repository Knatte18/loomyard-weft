# Repair 2: Discussion-Burler, history_length 6

Failure: `producer` error, not transient, on the re-step after repair 1.
The burler probe of round 2 found run dir `.lyx/shuttle/976793442de1fe3a22a34842438fe316`, left by the review strand `6e8ab264ee0d06be85628411512709bb` that the previous step stopped, and could not classify it because it was younger than 3m0s.

Trace lines acted on (`trace-20261008T062731Z-5cffcbfd923aa2a5-211891.log`):

- `shed: step refused kind=producer transient="" error="... probe the burler-review half: shuttle: attach: 1 candidate run dir(s) ... cannot be confirmed dead or alive (untracked or with a cleared pane binding, younger than 3m0s) ..."`

`lyx reed status` lists no strand `6e8ab264...`; the burler-fix strand `882569ccd79d25d0d75513828ebe5ab4` is still live.
No fabric mutation was half-finished.

Action: wait until the run dir passes the 3m0s age (08:28:43), no hand clearing, then re-step.
Outcome: see the stop report.
