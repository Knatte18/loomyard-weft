```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens over the diff's functions and error paths.

**Cases run** ("ok" means correct on that case)

| Function / branch | Case run | Result |
|---|---|---|
| `lastRepeats`, `DobbyRuns` fields, `readNoticeChild` | empty history; last entry with repeats 2 | ok |
| `LoadConfig` new refusals | `-0.5`, `0` min, factor `0` | ok |
| `LoadConfig` new refusals | `.nan`, `.inf` | F4 |
| `CheckLocalValues`, `checkTaskSetting`, `LookupRunWatch` | `all`, a bool, unset | ok |
| `runWatch` | nil seams; first `ReadState` fails; first record fails | ok |
| `runWatch` | second record fails after a post | F5 |
| `runWatch` unlisted entries | ahead of Land with and without landing memory; `StatusExists` error; owed line | ok |
| `watchRun` | level never read; level fails after a read; off; stops; rows; landed with land notice true, false or nil; no `Last` | ok |
| `readWatchedRun` | child error, not found, unchanged mtimes, zero mtimes | ok |
| `readWatchedRun` | child written between the run listing and the child read | F1 |
| `watchStop` | each derived state; child done reads derived `landing` | ok |
| `watchLine` | child reason, batten reason, child none | F3 |
| `postWatchLines` | no orch; third failure; hour of age; `NextTry` in the future | ok |
| `clearDoneRuns` | owed line; empty level; record failed; clear error | ok |
| `gateHolds` | config or holders fail; factor 0; post fails; record fails; two slots of one PID; non-probe tick | ok |
| `gateHoldProcesses` | `Tree` fails; marked PID already ended; `MarkedPIDs` fails | F2 |
| `gateHoldStart`, `gateHoldKey`, `gateHoldBound`, `gateHoldMarker` | mixed tiers; no tier | ok |
| `gateHoldLine`, `underWorktree` | empty tree; more than 20 processes; sibling prefix | ok |
| `dobbyGateHolders`, hub wiring | undeclared tier; broken config; no orch | ok |
| `runMarkerEntry`, `acquireSlot` | tier run, copies, detached runner | ok |

**Findings**

**F1. LOW, design.** Location: `internal/dobbyengine/runwatch.go` (`readWatchedRun`).
- **Summary:** one child stop can post two lines, because the derived state and the child part are read at different moments of the tick.
- **Case:** `DobbyRuns` derives `running` through `pairteardown.ReadRun`. The child then writes `blocked`, and `runWatch` later reads it, after the pass spawn, notices and landing notices. It posts `running; ... child blocked`. On the next tick the derived state is `halted`, so `reading.Run != last.Run` and a second line posts for the same stop. This breaks the first acceptance criterion, one line per block.
- **Evidence:** `internal/battencli/dobbyreads.go:203-222`, `internal/pairteardown/runread.go:218-228`, and the stops comparison in `runwatch.go` `watchRun`.

**F2. LOW, design.** Location: `internal/dobbyengine/gatehold.go` (`gateHoldProcesses`).
- **Summary:** a failed `MarkedPIDs`, or a failed `Tree` of one marked PID, throws away the holder tree that was already read.
- **Windows:** `PIDsWithEnv` always returns `errors.ErrUnsupported` there (`internal/proc/envmarker_windows.go`). Every tier run records a marker, so every tier gate-hold notice on Windows says `process tree unreadable` and names no child PID, although `TreeSample` works.
- **Against the key:** D6 asks for the holder's tree. Card 6's "a failed Tree or MarkedPIDs still posts, naming the tree as unreadable" matches the code, so the handler should weigh that.

**F3. LOW, design.** Location: `internal/dobbyengine/runwatch.go` (`watchLine`, `readWatchedRun`).
- **Summary:** `PairDir` comes only from a successful child read, so a batten reason is not made relative while no child status exists.
- **Case:** batten blocked at Seed-Child with a reason naming `<task worktree>/_lym/...`. `pairRelative(reason, "")` returns the text unchanged (`runnotice.go:476-478`), so the line carries the absolute path.
- **Against the key:** D3 says paths under the task worktree render relative to it.

**F4. NIT, design.** Location: `internal/dobbyengine/config.go` (the `GateHoldFactor < 0` check).
- **Summary:** `gate_hold_factor: .nan` or `.inf` passes the check. `gateHoldBound` then converts it with `time.Duration(...)`, which on amd64 gives the minimum int64, so every tier hold is noticed on the first probe.

**F5. NIT, design.** Location: `internal/dobbyengine/runwatch.go` (`runWatch`, the second `recordWatches`).
- **Summary:** posted lines leave the record only in one write after all posts. If that write fails, or the dobby dies between a post and the write, the next tick posts the same lines again. D4 asks that a restart repeat no line.

**Noted, not filed:** history length plus `Repeats` can collide. For example, 3+2 and then 5+0 after two new entries gives the same stamp, so a second block with no tick between would be dropped. The formula is the answer key's own (the added "Episode stamp counts folded repeats" entry), so the rubric bars a finding.

**Tool calls**
1. `pairRelative`, `oneLineCut` and `envmarker_windows.go`: the manifest held neither the helpers nor the Windows marker scan.
2. `runfsm.Derive` and the batten row names: needed to know what `RunView.Run` reads, which was not in the manifest.
3. The `runfsm` state order: the second call's output was cut before it.
4. `pairteardown/runread.go`, the landing and child inputs: one hop past `DobbyRuns`, to see when the derived state reads `landing` and when the child is read.
