# Plan review round 2 — fixer report

## Changes

- **F1 (BLOCKING), `_lym/plan/08-plugin-session-start-contexts.md`:** fixed differently.
  - Create gains `hookWaitDelay` (`const hookWaitDelay = 2 * time.Second`).
    The Intent says `cmd.WaitDelay` is `hookWaitDelay`, two seconds, beside the group kill.
  - The `TestRunPluginHook` timeout row now reads as the review proposed:
    `sleep 120; true` under a 100ms `Timeout` returns an error naming the timeout,
    and the call returns before a 60-second deadline that the test checks with a `select` on a done channel.
    No elapsed-time bound remains.
  - The group-kill row departs from the proposed `(sleep 2; touch $MARKER) & wait` with a 3-second wait.
    That proposal compares a measured real time to a bound a loaded machine can reach, which `PATTERN-test-speed` bars ("Time is injectable").
    Under load it passes without the group kill.
    The row now runs `sleep 120 & echo $! > $PIDFILE; wait` under a 5-second `Timeout`.
    It reads the pid and polls `syscall.Kill(pid, 0)` until `ESRCH`, under a 60-second deadline.
    Per that pattern, a pid is an outcome that tells the two paths apart.
  - Both rows stay in `pluginhook_integration_test.go`, which is tier1.

`lym loom validate-plan` reports `ok: true`, with only informational findings.

## Disputed

None.

## Deferred

None.

## Rulings

- **F2 (decision), cards 7 and 1:** parent decision #1559, "Unreadable-file test rows use a directory at the file's path".
  It asks that both unreadable rows of card 7's `TestUserSetup_InstructionFiles` and card 1's `TestExpand` unreadable-path row name a directory standing at the file's path as the input, with no root skip.
  Carried out:
  - Card 7's Tests now state that both unreadable rows put a directory at the file's path.
    `os.ReadFile` of it fails with an error other than `fs.ErrNotExist` for every user, so neither row skips itself under root.
  - Card 1's `TestExpand` row now reads: an unreadable `path`, a directory standing at it, returns an error other than `fs.ErrNotExist`, for every user, so the row never skips itself.

  Parent not told: the ruling was neither mistaken nor redundant.

## Self-check

- fix-consistency: no findings.
- fix-claims: no findings.
- fix-proposal, MEDIUM: the marker row decided the group kill from a 2-second versus 3-second race. Fixed with own change: the pid-based row described under F1.
  The lens also flagged that the pid file's write races the timeout. A 5-second `Timeout` for that row addresses this.
- fix-affected-set: no findings.
