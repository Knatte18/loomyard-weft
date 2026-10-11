# Webster round 1 — fixer report

All three findings were fixed, each in its own commit.
`lyx gate test ./internal/reedengine` and `lyx gate test --tags tmux ./internal/reedengine` pass after the last fix, and `lyx loom lint-comments` is clean.

## Changes

- **F1** (`734412246`), fixed as proposed.
  `tmuxConfPath` expands a leading `~/` only against a non-empty `home`.
  With an empty home the value stays `~/…`, fails the absolute check, and the error names the value the operator wrote.
  This is the behavior card 2 intended ("a `~/` path then fails the absolute check naming the path").
  `TestTmuxConfPath` gains the `HomeExpansionWithoutHome` row.
- **F2** (`d17de2c98`), fixed with the second branch of the proposal: end the server on the session-gone refusal when no reed session remains.
  `guardBootSessionLocked` now calls the new `endServerWithoutReedSessionLocked` before it returns the session-gone error.
  That function lists the reed sessions through `listSessionsArgv`, and on an empty or failed listing it runs `kill-server` and `ensureServerGoneLocked`, the same rule `Down` uses.
  It is best-effort: a failure is logged and never replaces the refusal.
  `doc.go`'s "Operator config" paragraph says that a refused boot ends the server too.
  Tests:
  - `TestTmuxConfBootGuard`'s `SessionGoneFailsTheBootNamingTheConfig` became two rows: no reed session left issues one `kill-server`, and another reed session listed issues none.
  - The integration subtest, renamed `FileEndingTheSessionFailsTheBootAndLeavesNoServer`, is now two rows: an untargeted `set destroy-unattached on`, and a file that runs `new-session -d -s operator` and then kills reed's session.
    Both rows assert that `ListSessions` reports no server after the refusal.
    The check runs before the key is registered, because `KillOnCleanup` pre-starts a kit server.
- **F3** (`3306d7d72`), fixed as proposed.
  The `tmux_conf` bullet now follows the `Socket`/`KillOnCleanup` pre-start bullet in `pattern/PATTERN-test-isolation.md`'s "Hermetic servers" section, with no blank line between bullets.

## Disputed

None.

## Deferred

None.
