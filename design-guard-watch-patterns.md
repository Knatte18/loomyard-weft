# Watch patterns on guard tests so a card gate runs only the guards its change can fail

`guard-watch-patterns` · Task · enhancement, gate, testkit · running · Open-Window

## Why (operator, 2026-11-08)

`docs/research/webster/webster-time-2026-11-08.md` measured the guard step of the card gate at 7,043 s of the card gate's 9,117 s over 14 Webster runs: 787 guard runs for 226 cards, about 8.4 min per run.
The operator keeps the guard step in every card gate, so a guard failure names the card that caused it.
The time is to come from running fewer guard tests per card instead.

## What

The gate already skips a watched guard test that a card's changes leave alone (`internal/gatecli/doc.go`, "Watched tests": a test marked `//lym:watch <pattern>...` is left out of the guard step when no changed file matches a pattern and its own file did not change).
Few guard tests carry a watch marker today.

- Read every `//lym:guard` test in the module and decide for each which files can change its outcome.
- Give a watch marker to each guard test whose outcome only a bounded set of files can change, such as the stencil guards in `contracts/stencils` over the stencil files; the patterns cover every file the test reads, so a change that could fail it is never skipped.
- Leave unmarked every guard test that scans all Go code or whose inputs cannot be bounded by patterns.
- The whole gates (plan-level verify, whole scope) run every watched test, as today, so nothing leaves the gate entirely.

## Report

In the change description, list each guard test with the patterns it got or why it stays unmarked, and estimate, from the 14 runs' changed files in the report above where they can be read, how many guard runs a card gate would have skipped.
