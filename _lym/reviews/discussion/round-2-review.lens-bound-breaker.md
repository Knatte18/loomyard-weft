```yaml
lens: bound-breaker
outcome: reported
```

Task: bound-breaker lens. I attacked every bound, guarantee and existing-code claim in the decision record.

**Claims attacked**
1. D1: running the watch after the landing notice step makes its last copy before Land the reading the landing notice decides with. **Held.** Each tick the landing notice step refreshes `Requested`, and `rememberRuns` decides from the memory without re-reading once the run is past Land (`landnotice.go:72-77`, `:119-149`).
2. D1: the watch reads `d.listedRuns` while `d.runs` drops done runs. **Held** (`passes.go:181-188`).
3. D1: the episode stamp (history length) separates a second stop at the same row, "as batten's notice keys tell episodes apart". **Broken.** See F1.
4. D2: a failed per-task read keeps the last level and warns once per episode, the way `land_notice` does. **Held** (`landnotice.go:82-88`).
5. D2 bound: "the setting only adds mail; it changes no gate, row or act rule". **Held** narrowly. An owed line holding the done-run clear is a clear delay, not a gate.
6. D3: the derived state reads landed once batten reads done, covering both landing-notice outcomes. **Held** (`runfsm/state.go:71-76`, `battenrecipe/land.go:27-42`).
7. D3: the landing notice decides and drops its memory at the latest on the first tick the run derives as landed. **Held.** `PastLand` is true from Worktree-Teardown on, so the decision comes on or before batten reads done.
8. D3: a cancel that removes its records leaves the list before any tick reads it, and "the watch posts no line for it". **Broken.** See F2.
9. D3: the cancel notice is tried once and a failure is only a Warn. **Held** (`cancel.go:829-836`).
10. D4: only the done-run clear and the watch drop an entry; `recordOwed` never touches the new map. **Held** by construction: the map is separate from `s.Runs`, which `passes.go:333-337` prunes.
11. D4: a reading equal to the memory sends nothing, so a restart repeats no line. **Held**, as long as the child part comes from the persisted memory when no status changed.
12. D5: untagged lines are never replaced and survive teardown. **Held.** `NotifyRunFrom` with empty `PostTags` sets no topic and no run (`notify.go:73-75`).
13. D5: three failed attempts at probe spacing, then drop. **Held as a stated bound**, though it is weaker than the landing notice's unlimited retry (`landnotice.go:158-159`).
14. D6: `Pool.Holders` drops a record whose lock is not held. **Held** (`pool.go:345-352`).
15. D6: a hold starts at the earliest `Started`, and a bound of factor × tier `timeout_min` on that start keeps the notice "on the same clock" as the tier timeout. **Broken.** See F3.
16. D6: the tier run's environment marker follows every process it starts. **Held** (`run.go:172`, `envmarker_linux.go:17`).
17. D6: the tier timeout's clock excludes suspend on Linux. **Held** as pinned. The 7-day sleep retention (`pauserecord/record.go:28`) also outlasts any plausible hold.
18. D6: a failed config load or `Holders` read skips the tick with one Warn per episode. **Held as stated.**
19. D6: the dedup key bounds a flood to one notice per hold. **Held** for a key of PID plus earliest start.

**Findings**

- **F1.** MEDIUM, design, `_lym/discussion/decision-record.md:40` (D1 episode stamp; also acceptance at :200)
  - Summary: the stamp is the history length, but a budget-exempt Stuck that recurs at the same row folds into the last history entry instead of appending. Block, resume, then block again at the same row with no tick between leaves state, row and length unchanged, so no second line goes. The acceptance bullet at :200 fails for that case.
  - Evidence: `internal/shedengine/run.go:698-720` (`appendOrFold`: a fold increments `Repeats` and sets `LastAt` without appending); `internal/shedengine/run.go:416-418` (a Halt Stuck is budget-exempt). Batten's own notice keys stamp by the episode's since time, not history length (`internal/battenshed/notice.go:143-145`), so the cited analogy does not hold.
  - Fix: make the stamp the history length plus the newest entry's `Repeats` (or `LastAt`). `ChildReading` already carries `NewestHistory`.
- **F2.** LOW, consistency, `_lym/discussion/decision-record.md:88-90` (D3, removing cancel)
  - Summary: a `--remove` cancel first writes its intent into batten's record (`Cancel` set, `Finish` nil), which `Derive` reads as cancelling. D3 counts cancelling as a stop, so any tick that falls while the cancel ends processes and removes the pair posts a `[run watch] … cancelling` line. That contradicts "the watch posts no line for it", and no cancelled line follows it.
  - Evidence: `internal/battencli/cancel.go:93-95` and `:377` (`WriteIntent`); `internal/runfsm/state.go:80-82` (cancelling while `Finish` is nil); `internal/dobbyengine/passes.go:39-41` (`CancelMarker`).
  - Fix: either state that a removing cancel can post a cancelling line, or exclude a reading with `CancelMarker` set or derived cancelling from stops. If the latter, record that the cancel notice then reports it.
- **F3.** MEDIUM, design, `_lym/discussion/decision-record.md:134-139` and `:160-162` (D6 bound and clock rationale)
  - Summary: the bound (factor × tier `timeout_min`, counted from the earliest slot's `Started`) is shorter than holds that are legitimate. Counterexamples:
    - A multi-slot tier holds slot 1 while it waits, unbounded, for the next slots. The tier's timeout only starts after all slots are held and the build has run.
    - The shared build runs inside the tier's slots with its own `timeout_min`. Build 20 + tier 35 = 55 min > 1.5 × 35 = 52.5.
    - `retryTier` reruns a crashed tier with a fresh tier timeout.

    Each case posts a false hung-gate notice. So the slot's age is not "on the same clock" as the tier timeout.
  - Evidence: `internal/gateslot/pool.go:264-297` (slots taken in order, each held while waiting); `internal/gatecli/run.go:650-662` (build, then `retryTier` inside `runWithSlots`); `internal/gatecli/run.go:705` (acquire before the body); `internal/gatecli/tierrun.go:124-135` (tier timeout starts in `runTierCommands`); `internal/gatecli/build.go:28-43` (build timeout inside the tier holds).
  - Fix:
    - Measure the bound from the latest `Started` of the hold's slots, and keep PID plus earliest start as the dedup key.
    - Size the bound to cover the build step's `timeout_min` and one retry, or state as an accepted bound that a build or retry can trigger the notice early.

Tool calls:
1. `cancel.go` `WriteIntent` and the tier timeout sites: the cancel's intent write and the tier timeout's start were not in context.
2. The probe default, `pauserecord.Retention` and batten's notice keys: needed to test the sleep retention and the episode-stamp analogy.
3. `run.go:640-720` and `tierrun.go:115-135`: the order of slot acquire, build, retry and tier timeout was not in context.
4. `shedengine` history appends: needed to learn whether a block appends a history entry.
5. `appendOrFold`: needed to learn when a Stuck folds instead of appending.
