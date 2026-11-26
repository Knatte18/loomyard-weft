```yaml
lens: walkthrough
outcome: reported
```

I walked each decision's mechanism through one concrete case (walkthrough lens).

**Decisions walked:**
- **D1:** a child blocks at row R, resumes, and blocks at R again between two ticks. The watch reads `d.listedRuns` in the run steps after `landNotices` (`loop.go:719`) and before `clearDoneRuns` (`loop.go:729`). The child mtime changes, the child is re-read, and the history length moves the stamp. Purpose met.
- **D1/D3:** a landed run with `land_notice` on. At tick T, still ahead of Land, `landNotices` stores `Requested` and the watch copies it. At T+1 the run reaches Worktree-Teardown and `rememberRuns` decides, posts and drops the line (`landnotice.go:142-177`); the watch's copy survives in its own map. When batten reads done, the derived state is landed (`state.go:71-76`) and the skip holds. Purpose met.
- **D2:** a task under `stops` whose child goes done, after which batten moves Run-Shed → Land → Worktree-Teardown. The child already reads Finalize-or-past, so the derived state stays landing, the stamp is the child's (unchanged), and no line goes. Purpose met.
- **D3:** a `--remove` cancel. The intent is written, the derived state reads cancelling (`state.go:80-82`), and the run is listed until `removeRecords` runs (`cancel.go:544-551`). A tick in that window posts a "cancelling" stop line. Purpose missed (F-w2).
- **D4:** a cancel that keeps its records. The run stays listed with batten cancelled, the line posts, and the entry persists; `clearDoneRuns` never takes the run, since it takes only `done` (`donerun.go:39`). Purpose met.
- **D5:** no orch strand recorded. `NotifyRunFrom` returns `posted=false, nil` through `postNotice`, so "waits and counts no attempt" is reachable. The existing `requirePosted` wrapper turns that into an error (`dobbydeps.go:89-96`), so the watch's `Post` has to keep the bool and not reuse the wrapper. That is plan-level, so purpose met.
- **D6:** a tier with `slots: all` (count 4) and `timeout_min: 35`. It takes slot 1 at t0 and waits 40 minutes for slot 2 behind another holder. Then it runs a shared build and a test run that crashes and is rerun. The hold reads as past its bound while the gate is healthy. Purpose missed (F-w1).
- **D7, D8:** prose and no change, so there is no mechanism to run.

**Findings:**

- **F-w1** — MEDIUM, design, `_lym/discussion/decision-record.md:134-139`
  - Summary: D6's bound and hold start measure time the tier timeout never bounds, so a healthy gate gets a gate-hold notice.
  - Start of the hold: the start is the earliest `Started` of the holder's slots. `takeSlotsInOrder` stamps each slot as it takes it and holds it while waiting for the next (`internal/gateslot/pool.go:270-298`, `:322`), so the earliest stamp includes the multi-slot wait. That wait can last a whole other tier's run, up to its own `timeout_min`.
  - What the hold contains: inside the hold, `runWithSlots` runs the shared build under its own `timeout_min` counted after the slots are held (`internal/gatecli/run.go:650-655`, `internal/gatecli/build.go:30-43`). It then runs `retryTier`, which reruns a crashed tier (`internal/gatecli/tierrun.go:402-410`).
  - Result: a lawful hold can exceed 1.5 × the tier's `timeout_min`, and the notice sent on it is false.
  - Fix: take the hold's start as the latest `Started` among its slots, which is when the slots are all held. Derive the bound from what a hold may lawfully contain: the build's `timeout_min` when the tier has a shared build, plus the tier's `timeout_min` per attempt including the rerun. Alternatively, state this bound as an accepted false-notice case.
  - Note: the hold-start sentence is a changed hunk from the F9 fix. The factor text predates this round.

- **F-w2** — LOW, consistency, `_lym/discussion/decision-record.md:88-90`
  - Summary: D3 says the watch posts no line for a cancel that removes the run's records, but a tick during the cancel reads the derived state cancelling, which D3 counts as a stop. So a removing cancel can still get a "cancelling" watch line before its status is deleted.
  - Evidence: `internal/runfsm/state.go:80-82` (cancelling while `Cancel` is set and `Finish` is nil), `internal/battencli/cancel.go:534-551` (records removed only at the end of the cancel), D3's stop list at `decision-record.md:76`.
  - Fix: either drop cancelling from D3's stop list, since the cancel's own notice and the cancelled line report the cancel, or state that a removing cancel can get one cancelling line.

Tool calls:
1. `postNotice`, `nearestOrchStrand` and `requirePosted`: the manifest lacked the no-strand return path that D5's "waits, counts no attempt" depends on.
2. `pauserecord.Retention`, the order of the build and slot acquire in `run.go`, and the gate config's `LoadConfig`: the manifest lacked the sleep retention, the build-inside-hold order and the gate config's scope.
3. `run.go:640-730` and `gateslot/config.go:178-192`: confirms the build and `retryTier` run inside the slots, which the manifest lacked.
4. `gateslot/config.go` `LoadConfig`, the `configreg` gate entry and the `retryTier` doc: confirms the gate config is hub-wide and that a tier reruns inside the hold.
