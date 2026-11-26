```yaml
lens: interactions
outcome: reported
```

Lens: interactions. I checked every pair of decisions that act on the same state, ordering or key, then the acceptance criteria and constraints against them.

**Pairs checked**
- D4 × the drop of a done run's state entry (`recordOwed`), with D3's landed line in force: **broken** (F1).
- D1 tick order × D3 landed/cancelled skip, against the landing notice's lifecycle: **broken** (F2).
- D2 per-layer "change into a stop" × D3 reading-level stop predicate (stop to stop, a batten resume while the child stays halted): **broken** (F3).
- D2 failed per-task read keeps the last level × D4 first-seen run with no memory: **broken** (F4).
- D1 file-change gate on the child read × D4 restart memory (re-exec, restart): held. After a restart the child is read again and an equal reading sends nothing.
- D1 absent child keeps its last part × the acceptance bullet on teardown: held.
- D1 episode stamp × D2 `stops` after the child's done (batten row moves, derived running→landing): held. Landing is no stop, and the child's history does not change.
- D4 clear hold × D5 drop bounds (three failures, one hour awake): held. Every owed line ends in a post or a drop, so the clear is bounded.
- D5 untagged lines × D6 own topic plus run tag × the acceptance bullet that a gate-hold notice and a run notice both stay unread: held.
- D2 `off` × D6 gate-hold notice and its acceptance bullet: held.
- D3 skip memory × D4 "a skipped reading still becomes the memory": held as written. It is moot given F2.
- D6 dedup key in the run's entry × D4 memory in the same entry: held for live runs. A done run holds no slot, so the drop in F1 does not hit this key.
- D7 orch role × D5/D8 (watch line first, then batten's detail as one fork): held.
- Constraint `PATTERN-daemon-wakeups` × D5 retry spacing and D6 probe tick: held.
- Acceptance bullet "Existing notice tests pass unchanged" (D8) × D4 extending the clear: held as stated.

**Findings**

F1 — BLOCKING, design, `decision-record.md` D4 (with D3, D5)
- **Summary:** D4 keeps the watch memory and owed lines in the run's entry of the state record. The existing pass step deletes that entry every tick for a done run with no pass in flight. A landed run, whose batten reads done, loses its owed landed line before the watch step runs. With the memory gone, D4's no-memory rule then treats the landed reading as new, so the landed line is posted again on every tick until the done-run clear removes the run.
- **Why D4 does not prevent it:** D4 only extends the done-run clear. It never mentions this drop, which `drivePasses` makes, not the clear.
- **Evidence:**
  - `internal/dobbyengine/passes.go:184-189`: done runs with no flight leave `runs` before `recordOwed`.
  - `internal/dobbyengine/passes.go:333-336`: `recordOwed` deletes `s.Runs[slug]` for every slug that is not listed.
  - `internal/dobbyengine/loop.go:710`: `drivePasses` runs before every other step of the tick.

F2 — BLOCKING, design, `decision-record.md` D3 landed/cancelled skip (with D1 ordering)
- **Summary:** The skip never fires for a landing. The landing notice's memory is decided at `PastLand`, which is true from the Worktree-Teardown row on. When `land_notice` is off, the memory is deleted at once. When it is on, the memory is deleted as soon as the line posts.
- **Timing:** The derived state reads `landed` only once batten's state is `done`, after teardown. By then the memory normally holds nothing, so a run with `land_notice` on gets both the landing notice and a `[run watch]` landed line.
- **Acceptance bullet broken:** "none when it reads requested".
- **Ordering:** D1 fixes the watch's place only relative to the done-run clear, not relative to `landNotices`. Even in the best order the memory is already gone.
- **Evidence:**
  - `internal/battenrecipe/land.go:27-29`: `PastLand` is true at `NameWorktreeTeardown`.
  - `internal/runfsm/state.go:71-75`: `Landed` holds only when batten's state is done.
  - `internal/dobbyengine/landnotice.go:142-149` and `:176-177`: the memory is deleted once decided off, or once posted.
  - `internal/dobbyengine/loop.go:719`: `landNotices` runs each scheduling tick.

F3 — MEDIUM, consistency, `decision-record.md` D2 `stops` vs D3 and the acceptance bullet "a resume to running … post nothing"
- **Summary:** D2 triggers a line on a change "into a stop" of any one layer (derived state, batten's state, child's state, episode stamp). D3 defines a stop over the whole reading. The two disagree in two cases:
  - A batten pause lifting while the child stays blocked moves the derived state from paused to halted. Under D2 that is a change into a stop, so a line goes, although it is a resume.
  - When the child goes blocked to failed with batten unchanged, the record does not say whether a change between two stop values counts as "into a stop".
- **Evidence:** `internal/runfsm/state.go:83-97`: Paused outranks Halted, so lifting a pause turns the derived state into halted.

F4 — LOW, consistency, `decision-record.md` D2 "A failed per-task read keeps the run's last level" vs D4 first-seen run
- **Summary:** A run first seen, as after D4's upgrade case, whose first `run_watch` read fails has no last level to keep. The record says neither which level applies nor whether its reading becomes the memory. The level and the memory together decide whether the upgrade line goes, and a later successful read with memory already set sends nothing.
- **Evidence:** `internal/dobbyengine/landnotice.go:72-91`: the existing pattern this copies has an explicit not-yet-read state (`Read`), which D2/D4 do not carry over.

**Tool calls**
- `grep recordOwed` in `internal/dobbyengine/passes.go`: the manifest showed the call site (passes.go:189) but not the body that deletes the run entries.
