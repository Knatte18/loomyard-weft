```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens, tracing the diff's changes to the decision record's D1 to D8 and its two added entries.

**Changes → decisions**

| Change | Traces to |
|---|---|
| `beam.run_watch`: key and values, config aliases, per-task check, `LookupRunWatch`, template line, tests (card 1) | D2 |
| `gate_hold_factor` and `gate_hold_min`: `Config`, range refusals, template, tests (card 1) | D6 |
| `PATTERN-config-layers` naming `beam.run_watch` | D2 |
| `boardcli` `run_settings` help line and its test; `configreg` rows | forced by D2 (the line's rewording to "are per task" comes with naming the fourth key) |
| `RunView` and `ChildReading` history, repeats and reason; `DobbyRuns`, `readNoticeChild` (card 2) | added entry "Episode stamp counts folded repeats", D1, D3 |
| `runwatch.go`, `State.Watches`, the tick call (card 3) | D1 to D5 |
| `donerun.go` holds and entry drop | D4, D5; the unread-level and failed-record holds come from card 3 and serve D2 and D4 |
| `RunWatch` hub wiring (card 4) | D5 |
| `Holder.Marker`, `runMarkerEntry`, `acquireSlot` (card 5) | D6 |
| `gatehold.go` and its tick call (card 6) | D6, added entry "Gate-hold line caps listed processes" |
| `GateHolds` wiring, `dobbyGateHolders`, `gateHoldTags` (card 7) | D6 |
| `orch-role.md` Notices bullets | D7 |
| `doc.go` and template descriptions | D7 |
| `docs/overview.md` dobby paragraph | forced by D5 and D6 (cards 4 and 7) |
| `TestTestOnlyCode` allowlist entry, added then removed within the range | forced by D2; no net change |

**Decisions → changes**

| Decision | Built by |
|---|---|
| D1 | `runwatch.go`, tick call, `RunView` fields |
| D2 | card 1, `watchRun` level handling, `PATTERN` line |
| D3 | `watchStop`, `watchLine`, land_notice copy (one partial gap, finding 1) |
| D4 | `State.Watches`, `runWatch` upkeep, `clearDoneRuns` |
| D5 | `postWatchLines`, `RunWatch.Post` wiring |
| D6 | cards 1, 5, 6 and 7 |
| D7 | `orch-role.md`, `doc.go`, `template.yaml` |
| D8 | no change to build; no existing notice, topic or hold is altered by the diff |
| Added: folded repeats | `LastRepeats` on both layers and the episode sum |
| Added: process cap | `gateHoldProcessMax` and the "; and <n> more" tail on each list |

No decision is left unbuilt, and no change is unrequested.

**Findings**

1. **LOW, design.** `internal/dobbyengine/runwatch.go` (`readWatchedRun`, about lines 233-235; `watchLine`, about line 282).
   - **Summary:** D3 says paths under the task worktree render relative to it, but `PairDir` is set only by a child read that finds a status. A run whose child was never read, such as batten blocked after Worktree-Create and before Run-Shed, renders batten's reason with absolute task-worktree paths.
   - **Evidence:** `readWatchedRun` assigns `reading.PairDir = child.PairDir` only inside `if found`. `internal/battencli/dobbynotice.go` `readNoticeChild` returns `found=false` with an empty reading when the child status is absent. `decision-record.md` D3: "Paths under the task worktree render relative to it". Card 3's wording ("relative to `PairDir`") matches the code, so the gap is against the record's D3.

Tool calls: none.
