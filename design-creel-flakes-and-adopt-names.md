# Fix two flaky creel tests and name failing tests in the adopt mail

`creel-flakes-and-adopt-names` · Task · bug, creel, gate · running · Finalize

## Seen

Three findings, one task: two flaky creel tests and the adopt check mail that left the first flake undiagnosable.

### Adopt check mail names the failing tests

Creel mail from `ly:ADOPT` reported `adopt check: tier0 failed, passed 21018 failed 2 skipped 1, 88.1s`, naming neither the failing tests nor their messages.
The names sit only in `suite.log` under `_board/.lym/adopt/`; `check.json` holds counts only.
The per-tier line is built in `runCheck` in `internal/adoptcli/check.go` from `CheckTier`, which carries no failure names.
The operator asked for this to be fixed and prioritized (2026-10-29).

### Daemon lifecycle test: "another daemon holds this hub" (issue #895)

A gate run failed once on the `internal/creelengine` `TestDaemon_Lifecycle` subtest covering a hook wait that reconnects across a daemon restart, with "the daemon ended before it listened: creel: another daemon holds this hub" (`server_test.go`).
No code under test had changed, and the identical gate passed on the next run.
This points to a race between the first daemon releasing the hub and the second daemon starting.

### `TestCreelCLI` EXTERNAL row (issue #901)

The `TestCreelCLI` EXTERNAL row failed twice in full-package tier1 runs while other worktrees' gates loaded the machine, then passed alone and in later runs.
It does not reproduce without load, so it looks timing-sensitive.
Telling a flake from a break took a stash round-trip to the base tree.

## Fix

- Carry the failing test names in `CheckTier`, and list each in the per-tier failure mail with its first failure line, plus the `suite.log` path.
- Look at the hub-release ordering in the creel daemon shutdown path, and make the lifecycle test wait for the release or retry the start, so it is deterministic.
- Find the timing assumption in the EXTERNAL row, a fixed wait or a tight deadline, and make it load-tolerant.
