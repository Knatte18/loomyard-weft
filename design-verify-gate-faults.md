# Darn verify gate: stop at the first failing tier, a time budget, and a truthful zero-attempt halt

`verify-gate-faults` · Task · bug, gate, darn, loom · draft

Depends on: [`gate-redundancy-report`](design-gate-redundancy-report.md)

## This task

Three faults of the verify gate a darn run's Darn-Write step runs, fixed together since they share its code path.

### Stop at the first failing tier

Seen (operator and orch, 2026-11-09): the Darn-Write verify gate of `arras-recipe-perch` failed tier 0 at 15:43:27 with 56 failures,
and the same gate started tier 1 at once and ran it for 11 minutes before it failed on two tests, then ran tier 2.
The first gate attempt took about 17 minutes; the tier 0 failure alone was known after 2.4 minutes.

When a tier fails, stop the gate there and report that tier's failures, so the fixer works on them before a slower tier runs.
Check first whether running every tier after a failure is deliberate, for instance to hand the fixer every failure at once,
and whether it differs between `lym gate test --upto` and the Darn-Write gate.
If a full report is wanted, make it a separate choice, and let the verify gate default to stopping at the first failing tier.

### A time budget

Seen (operator, 2026-11-09): the Darn-Write verify gate of `arras-recipe-perch` stood at 35 minutes, and that of `testmap-verb` at about 33 to 40.
`darn.verify_attempts` bounds the number of attempts, but each attempt is a full gate pass,
and nothing bounds the time a verify gate may take or tells anyone it is long.

Give the verify gate a time budget, and when it is spent, say so to the orch with the gate's failures to date,
so the operator sees it before it passes half an hour.
Decide whether a spent budget stops the attempts, hands the run to the orch, or only raises a notice.
Read first what a long verify gate shows today and where a time bound would sit:
in the recipe, in `darn.verify_attempts` beside it, or in the gate itself.

### A halt that names zero attempts

Imported from [issue #92](https://github.com/Knatte18/loomyard/issues/92).
A step halted as blocked with anomaly `escalation-to-human` and the message `gate did not pass after 0 attempts`,
pointing to a gate-findings file and `lym loom resume`.
It happened twice at the same producer step (`Darn-Write`), after the step was recorded `stuck` in the history.
Zero gate attempts means the gate never ran, yet the halt reads as a gate failure,
so the operator is sent to a findings file for a gate that did not run.

Make the halt message state why the step is stuck when no gate attempt ran,
and make the attempt count reflect the attempts made.
Provenance: task slug `slack-presence-auto`; notes `loom-halt-2.md`, `loom-halt-4.md`.
