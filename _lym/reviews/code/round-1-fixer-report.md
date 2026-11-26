---
dispositions:
  - id: F1
    disposition: fixed-differently
    reason: "Fixed as proposed, except that the path names .lym through lymdirs.DotLymDirName and not the literal the fix key asks for: PATTERN-lym-files lets only internal/lymdirs name .lym in a path. The constant is no engine method, so the fix key's aim still holds."
---

## What changed

All changes are in `internal/reedcli/entersession_integration_test.go`.

- F1: `TestEnterSession_DrivesTheToldTmux` asserts that `<anchor>/.lym/reed.lock` is absent before `enterSession`.
  After the existing checks it calls `stopWatchedSession` and asserts that the file exists.
  The `t.Cleanup` registration stays.
  The test's doc comment and the file header name the added behaviour.
- F1, differently: the path uses `lymdirs.DotLymDirName` in place of the proposed `".lym"` literal.
  `"reed.lock"` stays a literal, as proposed.

Commits: `d4b2fc222` (the assertion) and `31342c84a` (the self-check's two changes).

Checks:

- With the OOM watch temporarily back on a bare `go` in `watchdog.go`, the new assertion failed in 10 of 10 runs.
  `watchdog.go` was then restored and is unchanged in both commits.
- On the final code, `lym gate test ./internal/reedcli` (tier0) passed with 157 passed, 0 failed and 20 skipped.
- On the final code, `TestEnterSession_DrivesTheToldTmux` passed in 10 of 10 tier1 runs.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency, MEDIUM, `.lym` spelled as a literal in the new `opLock` line: fix as proposed.
- fix-consistency, LOW, file header not updated for the wait: fix as proposed.
- fix-proposal: no findings.
- fix-affected-set: no findings.

The fix-code-correctness and fix-proposal forks also noted the stale header outside their lens; it is the LOW above.

## Disputed

None.

## Deferred

None.

## Rulings

None.
