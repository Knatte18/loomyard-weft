# Burler round 2 — fixer report

## Changes

- F1 (MEDIUM), fixed as proposed: D1 in `_lyx/discussion/decision-record.md` now requires a kept value to survive every write path as the same JSON value, and states that the store's indented encoding may re-indent a nested value and escape a string's characters without changing the value.
  Premise confirmed: `state.WriteJSON` encodes with `json.MarshalIndent` (`internal/state/state.go:48`), which escapes HTML and re-indents.
  The number-digits clause is unchanged.
- F2 (LOW), fixed as proposed: D7's Bound in `_lyx/discussion/decision-record.md` now says the one-second path is reached while any process keeps adding entries to a dead directory, a concurrent sweeper's lock probe or a process outliving the dead owner, and is paid by every `newRun` while that lasts.
  Q7 in `_lyx/discussion/support-log.md` now says a persistent failure of another kind does not cost every `newRun` a second, and a persistent writer into a dead directory still does.

## Disputed

None.

## Deferred

None.
