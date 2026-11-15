---
verdict: CONTINUE
rationale: "Round 2 carries two BLOCKING design findings (F1 landing past the parent-side merge after a takeover, F2 old owner's code push before the records push), and F1, F4, F7 and F18 map to keys open in round 1 at MEDIUM or worse design or BLOCKING."
---

Round 2 raised 20 findings: 2 BLOCKING, 12 MEDIUM, 6 LOW.
Any BLOCKING finding forces `CONTINUE`.

Mapped to earlier keys:

- F1 reopens `landing-past-merge-stranded-under-hold`, now BLOCKING design: the carve-out holds on neither machine after a takeover.
- F4 maps to `cancel-removes-record-holding-its-intent`, MEDIUM design: the shed state of a kept cancel and its removers are unstated.
- F7 maps to `old-owner-mirror-outruns-adopter-restore`, MEDIUM design: the old owner's late mirror still wins.
- F18 maps to `d7-bound-understates-pre-read-activity`, LOW design (relabelled from the round 1 consistency class by the review's own entry), so it does not gate by itself.

No relabel of a BLOCKING finding.
No finding cites a declared PATTERN change entry directly; F3 touches `PATTERN-pair-remote` and is judged as an ordinary MEDIUM design finding.
The review has no Focus departures section.

F12 to F15 are `decision` class and go to the run's parent, not the fixer.
