```yaml
lens: code-correctness
outcome: reported
```

Task: run the code-correctness lens over the `usage-limit-hold-quiet` diff.

**Per-function list**

- `readTurnLimit`, newest is a `rate_limit` entry: returns `limited:true`, settled. Correct.
- `readTurnLimit`, newest is a plain turn end and the previous is found: `previousTurnEndAt` is parsed from the previous entry's timestamp and the reading is settled. Correct.
- `readTurnLimit`, only one turn end in the window: returns unsettled. `readBackward` widens until `start==0`, then stops with a zero `previousTurnEndAt`, so every fork counts. Correct. `foundNewest` is local, so repeated calls on larger windows restart cleanly.
- `readTurnLimit`, previous timestamp unparsable or empty: the parse error is swallowed, `previousTurnEndAt` is zero, and every fork counts. Matches the doc ("zero when ... its time does not parse"), but it can re-notice on an old limited fork. LOW, noted below.
- `readTurnLimit`, sidechain, non-assistant and unparsable lines: skipped. Correct. Mid-write partial last lines are skipped by the JSON failure.
- `TurnEndUsageLimited`, empty `Raw`, no `transcript_path`, missing file, or failed `Stat`: returns false. Correct. A zero-size file gives `readBackward(size=0)`, `scan(nil)` runs, nothing is found, false. Correct.
- `forkStoppedOnUsageLimit`, no `subagents` dir or `ReadDir` error: false. Correct. Non-regular and non-`.jsonl` entries are skipped. `Info()` errors are skipped. A fork mtime equal to `since` is excluded (`!After`), which is acceptable.
- `forkStoppedOnUsageLimit`, zero `since`: `ModTime().After(zero)` is true, so every fork is read. Correct.
- `newestAssistantIsUsageLimit`, the newest assistant entry is a limit entry and a later non-assistant line follows: skipped, true. Correct. Newest is a normal answer after an earlier limit entry: false. Correct.
- `newestAssistantIsUsageLimit`, no assistant line in the first window: the callback returns false and the window widens until `start==0`. Correct. A read error returns from `readBackward` with `limited` still false. Correct.
- `usageLimited` in `hold.go`: an engine without the capability gives `ok` false and no limit. Correct. `heldTurnEnd.raw` is set at both construction sites (`wait.go:851` and `wait.go:872`). A `grep` shows no others outside tests.
- The `Wait` switch, `continued` first: the continue case wins and `usageLimited` is not called. Correct.
- The `Wait` switch, limited: logs and does nothing else. The hold and wait mark (`beginHeldWait`) run before the switch, and `NotifiedOffset` is only written by `notifyHeld`, so it stays 0. Correct.
- The `Wait` switch, deferral queued by an earlier turn end and then a limited one: the new events read hits `run.pendingHold = nil` (`wait.go:300-303`) before the switch, and the limited case never sets `pendingHold`. The earlier deferred notice is dropped and cannot fire later. Correct.

**Findings**

1. LOW, design, `internal/shuttleengine/wait.go:327-329`. `usageLimited(held)` runs, and "no hold notice" is logged, for turn ends that would never have notified:
   - an interactive run (it holds without notifying);
   - `HoldDeferral != nil` with `turnStartRead` true (the next turn is already under way).

   The result is a transcript read per such held turn end and a misleading Info line. The decision record asks for the skip only on a turn end that would otherwise be notified. Order the check after those conditions, or fold it into the notice path.

   Evidence: `wait.go:322-334`. I did not confirm how `notifyHeld` filters interactive runs, so the interactive part is inferred.

2. LOW, design, `internal/shuttleengine/claudeengine/activity.go:255`. An unparsable or missing timestamp on the previous turn end silently becomes a zero bound. Every fork then counts, including one that stopped on the limit in an earlier turn, so a later notice can be silenced. The fork-recency bound is lost exactly when the timestamp is bad. Rare, and the doc states the fallback. Treat an unparsable previous timestamp as "use the previous entry's file position" or fail closed (no fork half).

   Evidence: `activity.go:255` (`reading.previousTurnEndAt, _ = time.Parse(...)`).

3. LOW, design, `activity.go:204-222`. The newest main-chain turn end is read from the live transcript, not the one the Stop payload names. If the next turn has already ended by the time `Wait` reads, the verdict is for the later turn end. This matches the decision record's "newest main-chain turn end", so it is a known limitation and not a deviation. It is only a window for a wrong skip or a wrong notice.

No other defects found in the remaining branches.

**Tool calls**
1. `grep` for `heldTurnEnd{` constructions and `isTurnEnd`: my context lacked a repo-wide search for other construction sites and the definition of `isTurnEnd`.
