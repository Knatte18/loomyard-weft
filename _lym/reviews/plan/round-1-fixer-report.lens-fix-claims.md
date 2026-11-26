```yaml
lens: fix-claims
outcome: reported
```

Task: check that each claim the fix added or rewrote holds against the code at HEAD.

One finding, LOW:

- **The new clear hold in card 3's Entry upkeep can hold a run forever.**
  - **Changed text:** "`clearDoneRuns` leaves ... while `RunWatch.Level` is set, a run with no level read yet (no `watchLevels` entry)".
  - **Summary:** a `Deps` with `Level` set and `Post` nil would keep every done run from being cleared.
  - **Evidence:** the card's Seams paragraph says "A nil `Level` or a nil `Post` sends no watch line and keeps no memory". If the watch step does nothing when `Post` is nil, `watchLevels` is never filled. The new hold checks only `Level`.
  - **Impact:** production (card 4) and the planned test rows set both seams, so nothing breaks today. Gating the hold on both seams being set would match the Seams rule.

Every other claim the fix added holds:
- **Card 5:** a nested run inside an inherited slot takes no slot and records no holder. It acquires only when `!e.inherited` (`internal/gatecli/run.go:696`), and line 172 replaces the marker entry with its own PID for every process it starts.
- **Card 1:** `CheckLocalValues` is the beam module's `CheckValues` (`internal/configreg/configreg.go:151`), so `TestPrepareRunSettings` reaches it. `storeVerbs` (`internal/boardcli/cli.go:558`) is the function that holds the `run_settings` help line.
- **Card 3, the landing notice never remembers a run first seen again past Land:** true per `rememberRuns` (`internal/dobbyengine/landnotice.go:131`).
- **Card 3, the watch step runs after `landNotices` and before the clear:** true per the tick order in `internal/dobbyengine/loop.go:709-729`.
- **Card 6, the failing-`LoadConfig` row now runs through ticks:** `gateHolds` comes before `startReady` in that same block, so the row's ticks reach it.

Tool calls:
- One grep of `internal/boardcli/cli.go`, `internal/configreg/*.go` and `internal/gatecli/run.go`. My context had not shown the function holding the help line, where `CheckLocalValues` is wired, or how an inherited slot skips the acquire.
