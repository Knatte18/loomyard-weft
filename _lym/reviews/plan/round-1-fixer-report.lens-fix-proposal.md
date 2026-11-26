```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals that the fixer carried out as proposed.

**Proposals checked:** F1, F2, F3, F4, F5, F7, F8, F10, F11, F12 and F13. F9 was fixed differently, and F6 is the parent's ruling, so neither is a proposal I own.

**[MEDIUM] F3's copy reset also fires for a run whose landing notice already posted, so it can send a landed line that should be skipped.**
- **Changed text:** card 3, Entry upkeep: "An entry whose slug is not listed this tick and that `State.LandNotices` holds no memory of has its `LandNotice` copy set to nil … such a run's landed reading is never skipped."
- **The case:**
  1. land_notice reads on, and the watch copies `true` ahead of Land.
  2. Tick N: the run is past Land but does not derive as landed yet, which D3 says happens "often a tick before". `rememberRuns` decides the line, `settleLandNotices` posts it, and `dropLandNotice` deletes the memory (`internal/dobbyengine/landnotice.go:142-149,177`).
  3. Tick N+1: the run is unlisted, for example because its status is unreadable. `LandNotices` holds no memory of it, so the reset sets the copy to nil.
  4. Tick N+2: the run is listed and landed. With a nil copy the landed reading is never skipped, so the watch posts a landed line beside the landing notice that already went.
- **What it breaks:** the acceptance criterion "A landed run posts … none when it is on".
- **The rule cannot tell the two cases apart.** "No memory" means either that the memory was dropped without a post (F3's own case) or that the notice has already posted (this case).
- **A sound alternative:** reset the copy only when the entry's last listed reading was ahead of Land. That needs a flag in `RunWatchState` set from `run.PastLand` on each listed tick. Then a run already past Land when the landing memory went keeps its copy.

**Other proposals hold:**
- **F12:** `LoadConfig` is called only in `starts.go:93` (`startReady`) and `tidy.go:165`, and both run after the gate-hold step. So driving the failing-LoadConfig row through `tick` reaches `gateHolds`.
- **F1:** `hubgeom.TaskSettingChain` skips a worktree that no longer exists (`internal/hubgeom/tasksetting.go:30-41`). So a torn-down task worktree does not keep `Level` failing and hold the clear forever.

Tool calls:
- One grep of the `LoadConfig` callers in `internal/dobbyengine` and of `hubgeom.TaskSettingChain`. My context lacked the tick's config callers (needed for F12) and how the chain treats a missing worktree (needed for F1).
