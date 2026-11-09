# Darn-Write gate fix: rerun only the failed tests between attempts

`gate-fix-failures-only` · Task · enhancement, gate, loom · draft

## Seen (orch, 2026-11-09)

The darn run of `testmap-verb` spent about 40 minutes in its Darn-Write verify gate: a first pass of 23 minutes over nearly the whole suite (tier 0 to 2), two failing tests, then a second full pass of 9 minutes, then a tier 1 rerun the gate started itself when the second pass ended with `exit_code 1` and no failing test.
The computed scope was close to the whole suite because the diff touches widely imported packages, and no coverage map exists yet to narrow it.

What exists today (`contracts/recipes/darn-recipe.yaml`, `Darn-Write`): the verify gate re-prompts the live session up to `darn.verify_attempts` times (3), and a cold fixer of `darn.gate_fixer` (`sonnet[medium]`) clears a failing verify gate before the writer hears of it.
Each attempt runs the gate over its full computed scope again.
The gate records each failure with its package, test and first line, and `lym gate test <package> -- -run <test>` runs chosen tests alone.

## Idea

Between fix attempts, re-run only the tests the last gate run recorded as failed, with `-run`, and hand the fixer only those failures; run the full gate once, after the fixer's changes pass alone.
That saves the full pass each failed attempt costs today, about 10 minutes each on this machine.

Read first, before any change: what the fixer and the writer receive on a failed gate (the whole log or the failures), and whether a rerun of failed tests alone is already possible inside the gate.
A failure with no failing test, such as a crashed runner or a build error, needs its own path, since there is no test to name.
