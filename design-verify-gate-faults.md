# Darn verify gate: stop at the first failing tier, rerun only failed tests, a time budget, and a truthful zero-attempt halt

`verify-gate-faults` · Task · bug, enhancement, gate, darn, loom · running · Drift-Verify

## This task

Four changes to the verify gate a darn run's Darn-Write step runs, made together since they share its code path.

### Stop at the first failing tier

Seen (operator and orch, 2026-11-09): the Darn-Write verify gate of `arras-recipe-perch` failed tier 0 at 15:43:27 with 56 failures,
and the same gate started tier 1 at once and ran it for 11 minutes before it failed on two tests, then ran tier 2.
The first gate attempt took about 17 minutes; the tier 0 failure alone was known after 2.4 minutes.

When a tier fails, stop the gate there and report that tier's failures, so the fixer works on them before a slower tier runs.
Check first whether running every tier after a failure is deliberate, for instance to hand the fixer every failure at once,
and whether it differs between `lym gate test --upto` and the Darn-Write gate.
If a full report is wanted, make it a separate choice, and let the verify gate default to stopping at the first failing tier.

### Rerun only the failed tests between attempts

Seen (orch, 2026-11-09): the darn run of `testmap-verb` spent about 40 minutes in its Darn-Write verify gate.
A first pass of 23 minutes ran over nearly the whole suite (tier 0 to 2) and two tests failed;
a second full pass took 9 minutes, and then the gate started a tier 1 rerun itself,
when the second pass ended with `exit_code 1` and no failing test.
The computed scope was close to the whole suite because the diff touches widely imported packages,
and no coverage map exists yet to narrow it.

What exists today (`contracts/recipes/darn-recipe.yaml`, `Darn-Write`):
the verify gate re-prompts the live session up to `darn.verify_attempts` times,
and a cold fixer of `darn.gate_fixer` clears a failing verify gate before the writer hears of it.
Each attempt runs the gate over its full computed scope again.
The gate records each failure with its package, test and first line,
and `lym gate test <package> -- -run <test>` runs chosen tests alone.

Between fix attempts, rerun only the tests the last gate run recorded as failed, with `-run`,
and hand the fixer only those failures.
Run the full gate once, after the fixer's changes pass alone.
That saves the full pass each failed attempt costs today, about 10 minutes each on this machine.
Read first what the fixer and the writer receive on a failed gate (the whole log or the failures),
and whether a rerun of failed tests alone is already possible inside the gate.
A failure with no failing test, such as a crashed runner or a build error, needs its own path, since there is no test to name.

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
