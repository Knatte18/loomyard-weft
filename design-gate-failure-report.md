# The gate reports every failure at once, lint included, in one readable result

`gate-failure-report` · Task · enhancement, gate, lint · running · Describe

## Seen (orch, 2026-11-01)

In the five heaviest Webster forks of the three latest tapestry runs, about 10 turns went only to reading gate output: the result was too large for one tool result and was saved to a file, or a `tail` showed only passing tests.
Lint ran in turns of its own after the gate (`lym loom lint-comments`, `lint-pattern`): 14 lint-only turns, and fixed-column comment findings cost about 8 more.
A fork fixed one failure, ran the whole gate again and read the next, rather than fixing every failure at once.

## Idea (operator, 2026-11-01)

The gate's report is written so that every failure can be fixed in one go:
- `lym gate test` prints one compact list of every failure: build errors, test failures and lint findings, the gate's own lint steps included.
- Each failure names its package, test and `file:line`, the expected and actual values or the first lines of the panic or build error.
- It names the files the change touches and the path of the full log, and stays small enough to read as one tool result.
