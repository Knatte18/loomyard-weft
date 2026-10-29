# The adopt check mail names the failing tests

`adopt-check-names-failures` · Note · bug, gate

## Seen

Creel mail #2 from `ly:ADOPT` reported `adopt check: tier0 failed, passed 21018 failed 2 skipped 1, 88.1s`.
It names neither the failing tests nor their messages, nor where the output is kept.

## Idea

The adopt check failure mail lists each failed test with its first failure line, and the path of the full gate output.
The operator asked for this to be fixed and prioritized (2026-10-29).
