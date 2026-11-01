```yaml
lens: existing-paths
outcome: reported
```

Lens: existing-paths. I traced every existing code path that writes, resets or reads the state the Decisions add or change.

**Paths traced, per decision**

- D4, `HoldCaller` entry, dobby's start step (`startReady` through `PauseCovers`): survives. A released task under a hold is not started.
- D4, dobby's owner-pause recheck (`internal/beamcli/dobbydeps.go:315`, which lifts only `OwnerCaller` entries): survives.
- D4, adopt's own lift of owner entries (`internal/adoptcli/adopt.go:735`, which lifts only `OwnerCaller` entries): survives.
- D4, the hibernate wake (`internal/hibernateengine/wake.go:99`, which records only its own entry): survives.
- D4, the orch pause watcher (`internal/orchengine/pause.go:86`, `handleHubEntry`, which handles hub entries only): survives. A slug-scoped hold interrupts no orch.
- D4, batten cancel's `liftSlugPauses` (`internal/battencli/cancelcmd.go:458-481`, which lifts every entry of the slug, holds included): survives. A cancelled run loses its hold, and D5's green end lifts only the entries that still stand.
- D4, dobby's `max_runs` slot count (`slotsTaken` and `runsInFlight` in `internal/dobbyengine/starts.go`): **breaks** (finding 1).
- D5, the green end's cancel through batten cancel, which writes `draft` through `clearRunClaim` and keeps the pair: **breaks** D8's start hold and D6's "never filed twice" (finding 2).
- D6 and D8, the board's run-agent task guard (`checkTaskCreation`, with `StrandName` set only by boardcli from `LYM_STRAND_NAME`): the record is silent on it (finding 3).
- D8, the `front` and `on_land` fields against the other board write verbs:
  - `upsert`: refused, as D8 states.
  - `upsert-batch` and `merge`: not covered, so the sole-writer claim **breaks** (finding 4).
  - rename: survives, because kept keys and every field are carried.
  - the owner write and the replay of a local commit: survive, because they copy entries (`PATTERN-tolerant-records`).
- D8, `on_land` remembered the way `notify_on_land` is (`internal/dobbyengine/landnotice.go`): **breaks** when dobby is down or restarts across the fix task's Land row (finding 5).
- D5, the check record's per-subject directory: its readers are only in adoptcli (`internal/adoptcli/checkview.go:39`, `internal/adoptcli/deps.go:151`, `internal/adoptcli/check.go:307`), and the record names the status bar and `lym beam status`. Survives.
- D7, the `paused` state read from the caller of another entry: survives.

**Findings**

1. **BLOCKING · design · D4 × D6/D8: held runs fill `max_runs`, so the fix task never starts.**
   - `slotsTaken` counts every listed run that the record does not mark owned elsewhere and that is not a pawl run without its landing hold. Pause-covered held runs are counted.
   - `startReady` stops at `running >= cfg.MaxRuns`.
   - The template default is `max_runs: 3`.
   - So a subject with three or more held moved runs never starts its released `adopt-fix-<subject>` task. The check never goes green, and no hold lifts.
   - The record is silent on how a held run counts toward the slot limit.
   - Evidence: `internal/dobbyengine/starts.go:142-165` (`runsInFlight`, `slotsTaken`), `internal/dobbyengine/starts.go:77-80`, `internal/dobbyengine/template.yaml:1`.

2. **BLOCKING · design · D5 × D6/D8: batten's cancel leaves the fix task a `draft` front entry.**
   - `clearRunClaim` writes `boardengine.StatusDraft` over a run status.
   - Without `--remove` the pair is kept, and the slug cannot run again until `lym batten cancel <slug> --remove`.
   - The entry still carries `front: true`, and it is neither done nor removed. D8's start hold "lifts once it is done or removed", so dobby starts no other task in the worktree after a green check.
   - D6's "a fix task of the subject that is still open is never filed twice" then finds the draft open, and files no fix after any later red check.
   - Evidence: `internal/battencli/cancelcmd.go:491-503` (`clearRunClaim`), `internal/battencli/cancelcmd.go:14-20` (the help on keeping the pair), `internal/battencli/cancel.go:418-434`.

3. **MEDIUM · design · D6/D8: the board's run-agent task guard is not addressed for fronttask's task write.**
   - `checkTaskCreation` refuses a new task when `Config.StrandName` parses as a slug-carrying, non-orch name.
   - Only boardcli fills `StrandName`, from `LYM_STRAND_NAME`.
   - D8 does not say whether fronttask's in-process `boardengine` write carries a strand name, or what lets the write through.
   - The check runs as a detached child of adopt, and of dobby's `on_land` spawn, so it inherits whatever strand name its spawner carries. Either way the guard is passed or refused by accident, not by design.
   - Evidence: `internal/boardengine/taskguard.go:13-36`, `internal/boardengine/config.go:55-57`, `internal/boardcli/cli.go:161`.

4. **MEDIUM · design · D8: `upsert-batch` and `merge` can still write `front` and `on_land`.**
   - D8 refuses the two fields only on `lym board upsert`.
   - `lym board upsert-batch` and `lym board merge` also upsert entries.
   - That breaks the declared `PATTERN-front-task` rule that fronttask is the sole writer of those fields.
   - Evidence: `internal/boardcli/cli.go:517` (`upsert`), `:569` (`upsert-batch`), `:851` (`merge`).

5. **MEDIUM · design · D8 × D6: `on_land` is lost when the parent dobby is down across the fix task's Land row.**
   - D8 copies `notify_on_land`'s memory. That memory is held only for a run first seen ahead of Land: "A run first seen past its Land row is never remembered".
   - A run no longer listed has its memory dropped. This is the known limit note `landing-watch-limits` records.
   - If the subject's dobby is down or restarts across the fix task's Land row, `adopt-check` never runs. The holds stand and nothing tells the operator.
   - Evidence: `internal/dobbyengine/landnotice.go:34-39`, and the board note `landing-watch-limits`.

Tool calls:
1. grep for readers of `.Caller`: to find every path that matches pause entries by caller.
2. `liftSlugPauses` and the board merge entry points: to see whether batten cancel lifts a hold.
3. The hibernate watch lift and the orch pause watcher: to see whether either reacts to a new slug entry.
4. The orch watcher's scope and the landnotice outline: to trace how dobby remembers an action past Land.
5. grep for readers of `AdoptScratchDir` and `check.json`: to judge the move to a per-subject check record.
