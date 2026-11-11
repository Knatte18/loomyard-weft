```yaml
lens: existing-paths
outcome: reported
```

Lens: existing-paths. I traced the code paths that already write or read each piece of state the decision record introduces. One path breaks a decision; the rest survive.

**Paths traced, per decision**

- **D2, the round-1 focus as the seed.** Every path that writes or archives the round-1 focus file keeps D2's rule intact:
  - `runSeedSpawn` archives it (`internal/shedadapters/bouncer.go:1746`) and `ensureFocus(1)` rewrites it (`:1009`). Both run only in the seed, before round 1's review starts. Survives.
  - The judge, replay and decision archives (`bouncer.go:1028`, `:1136`, `internal/shedadapters/bouncerdecision.go:183`) touch only round n+1 outputs, never round 1. Survives.
  - A clear (`bouncer.go:467`) and `startNextGeneration` (`internal/shedadapters/amendment.go:42`) call `archiveRunDir`, which carries the records over and leaves an empty directory, so the seed writes a new focus file. Survives.
  - An amendment with an allowance of 0 settles through `approveWithoutReview` (`amendment.go:56-57`). No round runs, so nothing is asked. Survives.
  - A fresh `lym loom start` writes a new seed, and the ask record sits outside the run directory, so the seeds differ and the round asks. Survives.
- **D2, the `history` reading.**
  - A goto appends an entry whose producer is the target row, at the goto time (`internal/shedengine/goto.go:140`). For a goto to Discussion-Write that entry is itself after the latest ask. Read right.
  - `appendOrFold`/`foldsInto` (`internal/shedengine/run.go:725-755`) fold only budget-exempt Stuck entries, so a Discussion-Write entry that ends Done is never folded. Survives.
  - Each row runs as a child step process (`internal/shedverbs/loopwatchdog.go:1`), which rebuilds the wiring (`internal/loomcli/wiring.go:612`, `:669`). So "each round reads the key" holds. Survives.
- **D3, routing.** `NotifyParent` routes through `parentOrchStrand` → `orchroute.FromPair`. The guard's `decisionParent` → `NearestLiveParent` resolves the same chain: the nearest live orch above the pair's parent, else the prime's. Survives.
- **D4, the copy beside the review.**
  - The review row archives one list, `paths` (`internal/shedadapters/burlerreview.go:276-280`), both on a failed exit and before each of its two in-row attempts (`:316`). The parent copy survives if it joins that list beside the assistant's copy, as D4 says.
  - The guard reuses `decisionCallerRefusal`, so the sampler strand is refused under `--by parent`. Survives.
- **D4/D5, the accepted-review hand-on.** A restarted review row that finds the round's review already accepted hands on to the fixer without running the table (`burlerreview.go:257-259`). **Breaks** — finding 1.
- **D5.**
  - The in-row attempt 2 starts after attempt 1's table ends with no review written, so the ask stays open and attempt 2's chair waits up to the first deadline. Survives.
  - The room is recomputed in each step's wiring, so a raised `review_timeout_min` applies from the next step. Survives.
  - The chair's await returns at each keep-warm cap, as the assistant's `lym shuttle await` does today, so the step watchdog sees activity. Survives.
- **D6.** The one fork deny in claudeengine matches only `lym webster` (`internal/shuttleengine/claudeengine/settings.go:301-306`). D6's claim that `lym loom` verbs carry no fork check holds. Survives.
- **D7.** No production code reads finding `origin` values: `internal/burlerengine/verdict.go:78` only declares the field. So `origin: parent` passes the review gate. Survives.

**Findings**

1. **MEDIUM — design — D5 (table-end close) and D4 (copy).** A restarted review row hands on without running the table, so the ask is never closed and a parent answer that arrives before the deadline is silently lost.
   - Evidence: when the round's review is already accepted, the row returns `p.handOn(...)` before any table runs (`internal/shedadapters/burlerreview.go:257-259`). D5's close ("once the review table ends with the review written, Go sets an ask still `pending-post` or `open` to `timeout`") and D4's copy both hang on the table ending, so neither runs on this path.
   - How it happens: the driver dies after the review is written but before Go closes the ask. On restart the row takes the hand-on path and the ask stays `open`.
   - Effect: a parent answer that arrives before the deadline is accepted as `done`, but no review holds its findings and no copy is committed.
   - Then D2's "same seed, round before, `done` with findings" rule asks the parent to confirm fixes Burler-Fix never made. If the Bouncer approves this round, the findings are lost silently.
   - The record's backstop, "an ask whose row never runs again is closed by D4's deadline check on the first late answer", covers only answers after the deadline. Here the row does run again, and the answer comes before the deadline.
   - This breaks D5's claim that "an ask the chair left without running the await is closed and a late answer to it is refused".
   - Fix: D5 states that the hand-on path also closes a still-open ask of the round, and D4 that it copies a `done` report. Or D5 runs the close right after the review is accepted, before any hand-on.

**Tool calls**

1. `grep` for fork refusals and the `archiveStaleOutputs`/`archiveRunDir` call sites. My context did not hold the writers of the focus file and the round outputs.
2. `sed` of `claudeengine/settings.go:285-325` and `shedadapters/burlerreview.go:255-330`. My context held neither the fork deny's scope nor the review row's archive, attempt and hand-on paths.
3. `sed`/`grep` of `shedverbs/loopwatchdog.go` and the call sites of `ResolveReview`/`segmentAssistBounds`. I needed to know whether config is re-read per step.
4. `grep` for `origin` in the burler engine. I needed to know whether the review gate checks the origin vocabulary.
5. `grep` for code that reads `.Origin` across `internal`. This confirmed that no gate reads finding origins.
