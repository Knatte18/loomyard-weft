```yaml
lens: bound-breaker
outcome: reported
```

Lens: bound-breaker. I attacked the record's bounds, ordering promises and claims about existing code.

**Claims attacked**

| Claim | Result |
|---|---|
| D1: `RunView.Run` is filled by `DobbyRuns` through `runfsm.Derive` | Held: `runfsm.Judge` with an empty `Aside` calls `Derive` (act.go:258-279, dobbyreads.go:206-207) |
| D1 rationale: a batten pause during the Land lock wait derives as running or landing | **Broken** (F4) |
| D1: an absent child status keeps the child part last read | **Broken** for a run that has no child status yet (F3) |
| D1: the watch runs before the done-run clear | Held: both run only on scheduling ticks (loop.go:709-729) |
| D2: under `stops`, batten's row moves after the child's done send nothing | Held: derived `landing` is not a D3 stop, and the stamp stays at the child's history |
| D2: a failed per-task read keeps the run's last level | Incomplete: there is no level before the first successful read (F7) |
| D3: the landed and cancelled skip reads the landing notice's memory, "so the two never disagree" | **Broken** in both directions (F1) |
| D3/Goal: a cancelled run is reported | **Broken** for a cancel that removes the records (F2) |
| D4: a reading equal to the memory repeats no line after a restart | Held |
| D4/D5: the done-run clear holds a run owing a line, at most one hour of awake time | Held |
| D5: `NotifyRunFrom` can tell "no orch strand" apart from a failed post | Held: `notifyPrime` returns `(false, nil)` when no strand is recorded (notify.go:263-266) |
| D5: untagged lines survive teardown and replace nothing | Held (notify.go:68-75) |
| D6: `Pool.Holders` drops records whose lock is not held | Held (pool.go:346-352) |
| D6: one notice per hold, keyed by slot, PID and start | **Broken** for multi-slot leases (F5) |
| D6: the tier run's marker needs a new holder-record field | Questionable: the marker is the holder's own PID (F6) |
| D6: the only failure skipped is a failed gate-config load | Incomplete: a `Holders` error has no stated path (F8) |
| D6: `gate_hold_factor: 0` sends nothing | Held; a negative factor is unbounded (F9) |
| D6: `lym gate status` already uses `Pool.Holders` | Held (gatecli/status.go:60) |
| D6: the clock claim (`CLOCK_MONOTONIC` excludes suspend) | Held as stated |

**Findings**

**F1. BLOCKING, design, decision-record.md D3 (skip rule)**
The landed and cancelled skip misfires both ways.
- **Landed:** `PastLand` turns true once batten's producer is Worktree-Teardown (battenrecipe/land.go:27-28). On that tick `rememberRuns` decides the notice, and `settleLandNotices` posts it and drops the memory (landnotice.go:142-149, 171-177). The derived state reads Landed only later, when batten's state is done (runfsm/state.go:71-75). By then the memory is gone, so the skip never fires and a run with `land_notice` on gets a second landed line. This contradicts acceptance bullet 7.
- **Cancelled:** a cancelled batten is neither done nor at Worktree-Teardown, so `PastLand` stays false. The memory keeps `Requested=true` while the run stays listed (landnotice.go:127-129). The watch then skips the cancelled line, and the landing notice never sends one either. A cancelled run with `land_notice` on reaches the orch by neither path, which breaks the Goal.

**F2. MEDIUM, design, decision-record.md D3 and the Goal ("cancelled")**
A cancel that removes the records takes the run out of the dobby's list before any watch reading sees it cancelled.
- `clearRunPhase` removes the pair and branch, then the records, all inside the cancel command (battencli/cancel.go:544-551). A plain cancel whose `FinishCancel` finds nothing also removes the records (cancel.go:552-564).
- `dobbyParentRuns` drops a run whose batten status is gone (dobbyreads.go:104-111).

The watch never reads `cancelled` for such a run, and the record does not say so.

**F3. MEDIUM, design, decision-record.md D1 and D3 (child part)**
Before Run-Shed a run has no child status: `ReadChild` returns `found=false` (runnotice.go:107-108). D1's keep-last rule covers only an absent child after teardown, with a prior reading. A newly started run therefore has an empty child state, which D3's "the child's state is not running" counts as a stop. Every new run would post a spurious stop line in the starting state, and D4's no-memory rule makes this happen on the first sight.

**F4. LOW, consistency, decision-record.md D1 rationale**
"A pause seen during the Land lock wait derives as running or landing" is false. `Derive` takes Paused whenever batten holds a `PauseIntent` or batten's state is paused (runfsm/state.go:83-85). The claim holds only for a batten block, because Halted reads only the child's state (state.go:87-97). The fix is to name a batten block as the example.

**F5. LOW, design, decision-record.md D6 ("once per hold, keyed by slot, PID and start")**
`AcquireSlots` takes several slots for one Holder with the same PID and Started (gateslot/pool.go:179-201), and there is one holder record per slot (pool.go:338-365, 411). A multi-slot tier hold therefore posts one gate-hold notice per slot. The key should be PID and start.

**F6. LOW, scope, decision-record.md D6 (new marker field)**
The tier run's marker is `LYM_GATE_TEST_PID=<pid of the lym gate test>` (gatecli/run.go:172, processcap.go:143-150, watchdog.go:23-25). `prepare` stamps the holder with the acquiring process's PID (pool.go:213-215). When the acquirer is the `lym gate test` process, the marker can be derived as `LYM_GATE_TEST_PID=<Holder.PID>`, so the new slot-record field is not needed. The record should either derive the marker or state why the acquirer's PID differs.

**F7. LOW, design, decision-record.md D2 ("a failed per-task read keeps the run's last level")**
When the first read of a run fails there is no last level to keep, and the record does not say which level applies until a read succeeds.

**F8. LOW, design, decision-record.md D6 (failure paths)**
`Pool.Holders` returns an error for the whole list when any held record fails to read or parse (pool.go:353-362). One garbled or half-written record therefore blanks the check for every run. D6 names only a failed gate-config load, so the `Holders` error path has no stated handling or Warn episode.

**F9. LOW, design, decision-record.md D6 (`gate_hold_factor`)**
Only a factor of 0 is given a meaning. A negative factor makes the bound negative, so every hold is noticed as soon as it starts. Nothing in the record validates the range of either hub key.

**Tool calls**
1. `battencli/cancel.go`, lines 534-606: to see whether a cancel removes the run's records, which my context lacked.
2. `modTimeOf`, `notifyPrime`, the callers of `Holders()`, and the holder-write lines in `pool.go`: to check the absent-file handling, the no-strand return, the `gate status` claim and the record write.
3. `gatecli/processcap.go`, lines 1-80: to find where the tier marker is formed.
4. The callers of `newProcessCap` and `runMarker`, and the holder write in `pool.go`: to find the marker's source.
5. `runMarker`, its tier-run caller, `AcquireSlots` and `prepare`: to check the marker's value and the multi-slot holder records.
6. The uses of `enclosingGateEnv`: to learn the marker's value, which is the PID of the `lym gate test` process.
