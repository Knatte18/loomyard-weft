# The adopt check mail names the failing tests

`adopt-check-names-failures` · Note · bug, gate

## Seen

Creel mail #2 from `ly:ADOPT` reported `adopt check: tier0 failed, passed 21018 failed 2 skipped 1, 88.1s`.
It names neither the failing tests nor their messages.
The names sit only in `suite.log` under `_board/.lym/adopt/`; `check.json` holds counts only.
The per-tier line is built in `runCheck` in `internal/adoptcli/check.go` from `CheckTier`, which carries no failure names.

## Idea

Carry the failing test names in `CheckTier`, and list each in the per-tier failure mail with its first failure line, plus the `suite.log` path.
The operator asked for this to be fixed and prioritized (2026-10-29).
