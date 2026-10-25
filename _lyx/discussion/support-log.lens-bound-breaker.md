```yaml
lens: bound-breaker
outcome: reported
```

Bound-breaker lens: I attacked every bound, guard and claim in the umbrella-daemon decision record.

**Findings**

1. **BLOCKING — D4 / D13: dobby's passes trigger themselves.**
   - D4 runs a pass whenever batten's own status file is written, and a pass writes that file. Shed rewrites `current_producer`, `state`, `activity` and `history` on every persist (`internal/shedengine/doc.go`).
   - Counterexample: a pass resumes Run-Shed, persists the status, checks once and yields. fswatch then reports the write and dobby starts another pass.
   - Each idle run therefore spawns one `lyx` process a second, the D13 ceiling, and never backs off.
   - The record has no rule that skips a write the pass made itself.

2. **BLOCKING — D5: dobby restarts a cancelled task.**
   - D5 starts any task whose status is empty. `lyx batten cancel <slug>` ends a run by clearing its status (`boardengine` doc, run lock section), and a hand `set-status` clear does the same.
   - Counterexample: the operator cancels a run without `--remove`. On its next board tick dobby sees an empty status with every dependency done and starts the task again.
   - Cancel no longer stops anything. The record never says what status a cancel leaves.

3. **MEDIUM — D5: a refused start is never retried after the cause is fixed.**
   - D5 retries only when the entry's `revised` time changes, but "a status change revises nothing" (`boardengine` doc).
   - Many refusals have causes outside the entry: another machine's owner, a beam-marked pair, a left pair, an unreachable origin.
   - Counterexample: `lyx adopt` or a cleanup fixes the cause and the entry is unchanged, so the task stays unstarted for good. Even a `set-status` toggle does not bump `revised`.

4. **MEDIUM — D4: the yield claim contradicts itself.**
   - D4 says a yield counts against no repair cap. It also says every other end of a pass is "the same as a batten process's end today".
   - Today a batten that exits mid-step is an interrupted step, which `sheddriver` counts against its repair cap.
   - If the yield is a plain process exit, every pass is an interrupted step and the cap trips within a few passes. The record names no mechanism that tells a yield apart.

5. **MEDIUM — D8: darn's arbiter values change silently.**
   - Darn reads its arbiter keys from the child's loom config today: `battenshed/deps.go` says "read from the child's loom config", and `arbiterstop/holdreading.go` reads `cfg.ArbiterMaxSpawns`. `darnengine/template.yaml` has no arbiter keys.
   - Counterexample: a hub whose `loom.yaml` sets `arbiter_max_spawns: 0` or another model also governs darn today. Under D8, darn inherits `shed.yaml`, so darn's arbiter turns back on or changes model.
   - That breaks "keeps the same meaning … no hub value needs moving".

6. **MEDIUM — D7 / D9: the orch hears of a failure before the arbiter.**
   - D9 keeps the Land and Worktree-Teardown failure notices unchanged. Today the row notifier posts them on every failure episode (`battenshed` doc, row notice).
   - D7 and D8 send batten stops through the tackler and the arbiter first, and only the child's notices have an arbiter hold, not batten's own row notices.
   - Counterexample: Land fails, the orch gets the notice at once, and the arbiter then settles the stop. "Arbiter first, then orch" does not hold for batten's own rows.

7. **MEDIUM — D4 / D9: the record claims every bound survives, but latencies grow.**
   - Between passes nothing watches the files that end several bounds: the stop report, the driver strand's end, the decision record, the three-minute report wait and the ten-minute relay wait. They are seen only at the next probe tick.
   - Counterexample: the driver writes its stop report at t+10s and the notice goes at the next probe, not at the next `poll_interval_s` check.
   - So "every … grace rule keeps its meaning" is false by up to one `notice_probe_s` per bound. The record should state that new bound.

8. **MEDIUM — D10: the tidy agent's parent is dobby, which cannot answer.**
   - `PATTERN-parent-directive` renders a parent the agent asks by `SendMessage`, and dobby has no LLM.
   - Counterexample: the tidy agent hits an ambiguous note and asks its parent, and the question goes nowhere.
   - D10 also says "ended … after its timeout" but names no timeout key or value, so that bound is unbounded as written.

9. **MEDIUM — Constraints: the no-migration evidence goes stale before the binary lands.**
   - The "no statusless open task" claim is a snapshot from 2026-10-25.
   - Counterexample: the orch upserts or promotes tasks with today's binary while this run is in flight, and those get no status. When the new binary deploys, dobby starts them at once, without the operator's approval that today's policy requires.
   - The Open risks section covers only other hubs, not this hub between now and the landing.

10. **MEDIUM — D3 / D5: two machines can start the same unowned task.**
    - D5 starts a task whose owner is "this machine or none". If two machines each run a dobby for a beam, or a prime, both read an unowned ready task.
    - Batten's fresh start pulls the board and refuses only a task another machine already owns. Two pulls before either claim both pass.
    - D2 does not say the prime's dobby runs only on the prime owner's machine. "Every beam worktree this machine owns" leaves the prime ambiguous.

11. **LOW — D5: dobby reads only the local board.** A draft cleared and pushed from another machine is not seen until something pulls `_board`. "Dobby reads it on a write to `board.json`" never fires for a remote change.

12. **LOW — D3: `lyx beam land` still launches through `battencli.ResumeInWindow`.** D4 retires `--window`, and the record does not say the beam landing now hands its run to the parent's dobby.

13. **LOW — D5: an older binary bypasses the draft default.** A pre-upgrade `lyx` still running, or one on another machine, creates tasks with an empty status, and dobby starts them.

14. **LOW — D11: deletions are unspecified.** The record does not say what the notice reports when the tip deletes a key or a whole file the parent still holds.

15. **LOW — D2: a failed `lyx beam start` is only a Warn.** That leaves runs with no dobby while `lyx orch start` reports success, against "a run's parent always has a running dobby".

**Claims attacked**
- D1 teardown ends dobby under the beam slug: held. `pairteardown` calls `runproc.End(slug)`, sparing the caller's ancestors, and the parent dobby is not under the beam slug.
- D1 nothing that ends member processes reaches dobby: held. Tender's orphan reap never treats `HubOwner` or a slug with a live worktree as orphaned.
- D1 re-exec keeps the lock: held, same pid.
- D2 idempotent: held. Prime ownership: broken (10). Failure only warns: broken (15).
- D4 yield spends no cap: broken (4). Self-trigger: broken (1). No double pass through the held driver lock: held. Bounds keep their meaning: broken (7). Watched marker holds dobby's pid: held.
- D5 starts only drafts cleared by hand: broken (2, 9, 13). Never twice while a pass is in flight: held on one machine, broken across machines (10). Retry on `revised`: broken (3).
- D8 old `loom.yaml` values keep their meaning: held for loom, broken for darn (5). Arbiter before orch: broken for batten row notices (6).
- D9 kept notices unchanged: held. api-error comes one probe later: held.
- D10 one agent at a time: held. Timeout bound: broken (8).
- D11 silent cases: held. Deletion: unspecified (14).
- D12 a pass after a lift: held. A Land halted blocked under the pause still needs `lyx batten goto` by hand, which predates this design.
- D13 at most one tick a second: held per event, but feeds (1).

**Further tool calls**
- Grepped the darn template and the arbiter key readers: needed to know where darn's arbiter keys come from today.
- Grepped `runproc.End` callers and batten's probe config: needed to know who can end dobby's owner records.
- Grepped shed's doc for persist and interrupted-step rules: needed to know whether a pass rewrites the status file.
- Read the `tendercli` deps around the `EndOwner` and owner-pause end calls: needed to confirm which owners tender ends.
