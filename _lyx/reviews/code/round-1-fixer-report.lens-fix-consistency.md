```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix-consistency lens on the change set (`holdNoticePending` gate in `wait.go`/`hold.go`, two new `TestTurnEndUsageLimited` rows).

**Findings**

- **LOW** — `internal/shuttleengine/wait.go` log line `"held turn end stopped on the usage limit; no hold notice"`, with its comment above it.
  - Summary: the narrowed case still agrees with its log line. The line now fires only where a notice would otherwise go, so F3's misleading-log premise is resolved.
  - Evidence: `doc.go:104` says a limit-stopped held turn end sends no notice, defers none and leaves `NotifiedOffset` alone. That still holds for every run where a notice was possible.
  - The cases the new guard excludes (interactive, `QuietHold`, covered, no notifier, `HoldDeferral` set with `turnStartRead`) fall through to the later `switch` arms. Those arms send or queue nothing for the first four. For the last one, the `!run.turnStartRead` arm is also false, so nothing is queued.
  - One difference remains. An excluded run now never reaches `usageLimited`, so nothing is logged for it. That is the intent.
  - Not a defect; recorded as "checked, consistent".

- **LOW** — `internal/shuttleengine/hold.go`, `holdNoticePending` doc comment and body.
  - Summary: the guard duplicates the skip conditions of `notifyHeld`, so the two can drift. `notifyHeld` has one condition the guard omits, `held.offset <= run.state.NotifiedOffset`. That omission is acceptable: an already-notified offset gets a harmless extra log line and read.
  - Evidence: `hold.go:153`, `notify == nil || run.covered || run.spec.Interactive || run.spec.QuietHold || held.offset <= …`. The guard repeats the first four and drops the offset check.
  - `notifyHeld`'s doc comment lists the skips. `holdNoticePending` has no comment tying it to them, so a skip added to `notifyHeld` later would not reach the guard.
  - Not a rubric defect; the stale-risk is minor.

- **No finding** — the new test rows.
  - `earlierTurnEnd, limitTurnEnd, agentTurnEnd` gives want false. The newest main-chain turn end is plain, so `reading.limited` is false. No fork exists, so `forkStoppedOnUsageLimit` is false.
  - `earlierTurnEnd, agentTurnEnd, sidechainLimit` gives want false. `readTurnLimit` skips `IsSidechain` entries, and `agentTurnEnd` is the newest main-chain entry.
  - The row names follow the table's sentence style and the constants sit with the sibling constants.

- **No finding** — docs. No other file describes the notice condition: `spec.go`'s `QuietHold` comment and `doc.go:91` are unaffected by the guard.

Tool calls (1):
1. `grep` for usage-limit mentions across `doc.go`, `spec.go`, `PATTERN.md` and `docs`: my context did not show whether a doc states the notice condition the guard narrows.
