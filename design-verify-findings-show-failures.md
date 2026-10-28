# Verify-gate findings log tail shows only passing tests, hiding the failure

`verify-findings-show-failures` · Note · bug, loom, gate

Imported from [issue #884](https://github.com/Knatte18/loomyard/issues/884).

## What happened

A producer row's verify gate failed three times in a row, and the run halted `blocked` with anomaly `escalation-to-human`.
Each re-prompt carried a findings text of the form "The verify command failed with exit code 1", a full-log path and a "Log tail".
The tail was the last lines of the `lym gate test` output, and those lines held only `--- PASS` entries, a final `PASS` and `ok` for the last package.
The `--- FAIL` lines sat tens of thousands of lines earlier in the same log (the verify log ran to roughly 57,000 lines, and the first failure sat near line 25,800).
The only hint of failure in the tail was the gate summary line `tier0 computed ... failed 2 ...`, which names no test.

## Why it is abnormal

The tail exists so the re-prompted agent can see what failed.
With `go test` output interleaved from parallel packages, the last lines are not where failures are, so the agent got a findings text that looks like a pass.
The agent had to open a 57,000-line log and search for `--- FAIL` by itself, and the budget of three attempts was spent re-running with little to act on.

## What to do

Build the findings text from the failures rather than from the raw tail.
For `lym gate test` output, extract the `--- FAIL` and `FAIL <package>` lines (with a few lines of context each) and put them first.
Fall back to the tail only when no failure marker is found.
Alternatively, make `lym gate test` print a failed-tests section at the end of its output, so any tail contains it.

Provenance: task `hub-reader-creates-lym`; notes `loom-halt.md`, `loom-halt-2.md`.
