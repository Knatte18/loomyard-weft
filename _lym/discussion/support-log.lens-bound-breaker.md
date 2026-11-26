```yaml
lens: bound-breaker
outcome: reported
```

Task: bound-breaker lens. I attacked the decision record's bounds, guarantees and claims about existing code. Five held, four broke, and three more are weak.

**Findings**

1. **MEDIUM. D4 and D1: a second stop at the same row can go unreported.**
   - The dedup key is the reading alone: derived state plus batten's and the child's state and row. It carries no episode stamp.
   - Counterexample: the child is blocked at Code-Burler-Fix and the watch tells it. The arbiter resumes the run, and the row fails again within a second.
   - The dobby ticks at most once a second and reads the child only when a status file changed. If no tick samples the brief running reading, the new reading equals the remembered one, so no line goes.
   - Batten's notice avoids this: its `noticeKey` holds the episode's since. The watch key should carry an episode stamp too, such as the child's history length or the status mtime at the first sight of the state.

2. **MEDIUM. D5: lines are dropped well before an orch restart ends.**
   - D5 retries "on the next tick, up to the run notices' three attempts". The run notices retry on probe ticks. Dobby ticks can be one second apart while runs are in flight.
   - `requirePosted` errors when no orch strand is recorded or live, since `nearestOrchStrand(…, OrchLive)` resolves it. So an orch that is down for more than about three ticks loses every stop line.
   - That breaks the Goal's "survives the orch's context cycles". Retries need time spacing, as batten's `noticeProbe` has, or the line must be held until a strand exists.

3. **MEDIUM. D6 against D5's own rationale: the gate-hold notice can replace a run notice, or be replaced by one.**
   - D6 tags the gate-hold notice "as the run notices are". `runNoticeTags` sets `Topic: runNoticeTopicPrefix+slug`.
   - `PostTags.Topic` returns undelivered the sender's unread mail under the same topic. An unread gate-hold notice is therefore replaced by a later driver-dead or usage-limit notice of the run, and the reverse also happens.
   - D5 rejects topic replacement for exactly this reason. The gate-hold notice needs its own topic, or none.

4. **MEDIUM. D4 and D5 against `clearDoneRuns`: the landed line can be lost before its retry.**
   - `clearDoneRuns` (loop step 7) clears a done run once its pair is gone and `current.LandNotices` holds nothing of it. It does not check the watch memory.
   - With `land_notice` off, which is the only case D3 posts a landed line in, a landed line whose post failed can be cleared the same tick. The run then leaves `listedRuns`, so the retry never happens.
   - The clear should also wait until the watch memory holds the run's final reading as posted or dropped.

5. **LOW. D2: "refuses the dobby's lookup … and the run reads as `stops`" contradicts itself.**
   - The dobby doc says a failed per-task read "keeps the last reading" for `land_notice`.
   - D2 should say which applies: keep the last reading, or fall back to `stops`.

6. **LOW. D6: awake time over "the dobby's recorded sleeps" misses suspends the dobby did not see.**
   - A sleep that ends while the dobby is down, or before it starts, records no interval. The slot's age then counts wall time.
   - Result: a spurious gate-hold notice, such as #215's overnight case if the dobby restarted after the wake.

7. **LOW. D6: `Pool.Holders` takes each free slot's lock briefly.**
   - `lockIsHeld` calls `TryAcquireWriteLock` and releases at once.
   - Every beam's dobby now does this on each probe tick, in addition to `lym gate status`. A concurrent `Acquire` can see a free slot as busy for that moment.
   - This is harmless if `Acquire` retries, but the record states no bound on it.

8. **LOW. D6: "lies under that run's task worktree" must be path containment, not a string prefix.**
   - `Holder.Worktree` is `location.WorktreePath()`. A naive prefix match would let task `x`'s worktree claim holders from sibling worktrees `x-weft` or `x-2`.

9. **LOW. D4: the first sight after an upgrade sends one line for every run standing at a stop.**
   - Several long-halted runs then mail the orch in one burst. A hibernate also pauses every run and sends one paused line each.
   - This is noise, not breakage, but neither the Open risks nor the bounds say so.

**Claims that held**

- #207 was fixed by `77630a4f1`.
- `Holders` drops records whose lock is not held.
- A batten blocked at Land derives as Landing in `runfsm.Derive`.
- The tier timeout runs on Go's monotonic clock, which excludes suspend. Starting the slot-age clock at `Started` is no earlier than the timeout's own start, so the bound is generous, not short.
- D7's "no second fork" fits the orch's existing per-slug fork rule.

**Tool calls**

1. Read `internal/gateslot/pool.go` lines 330–400, for how `Holders` tests a lock. My context had only the `Holder` struct.
2. Read `internal/dobbyengine/donerun.go` and grepped the tick order in `loop.go`, for when a done run is cleared relative to the notices. My context lacked both.
3. Grepped the `Worktree:` assignments in `gateslot` and `gatecli`, for what `Holder.Worktree` holds. My context had only the field comment.
