```yaml
lens: bound-breaker
outcome: reported
```

Bound-breaker lens: I attacked every bound, guard and existing-code claim in the decision record.

**Findings**

1. **BLOCKING — D4, D5: the card fixer's own re-run joins the run that is waiting for it.**
   - D5 tells every fixer to "re-run … the whole gate call". For the card gate that is the same `lym gate test --gate card --card <id>`, run in the same worktree.
   - The detached run's key is `detachKeyDir(anchor, call.verb, call.dir, call.words)` (`internal/gatecli/detachfront.go:304,364`). It hashes the verb, the call's directory and its words. No strand identity goes into the key, and "Calls with the same key share one current run."
   - Counterexample: the fork's red call starts the fixer inside run K. The fixer runs the identical call, gets the same key, and joins K. K is blocked waiting for the fixer's reply, so the fixer polls `done: false` until its 30-minute timeout and dies with `timeout`. Every card fixer deadlocks this way.
   - The `gate-fixer` role guard in D4 stops a nested spawn, not this join. The record needs the fixer's call kept out of the caller's run, for example by adding the strand or role to the key, or a fixer-only flag.

2. **MEDIUM — D1 (restart attach) and D2 (row callers): after a driver restart, the attach does not happen until the row's deadline.**
   - The writer is idle at its boundary when the fixer is spawned, so it sends no new arrival.
   - On `AttachGated`, `gateAtBoundary` is false. The loop then only asks the entries' `Expired` probes (`wait.go:33`, `wait.go:373-380`). It evaluates a closure only after a gated Done arrival or while a pending entry or unsent re-prompt exists.
   - Counterexample: the driver dies while the Darn-Write closure is blocked. On restart the writer stays idle, nothing evaluates, and the wrapped closure never reattaches. The fixer finishes or times out unseen. The run then sits until `writer_timeout_min` (180) or `discussion_timeout_min` (480), when the finalize evaluation re-runs the closure. At that point a still-failing gate halts the run instead of re-prompting the writer.
   - The acceptance line "a caller restarted while its fixer is live attaches to that strand" holds only in that degenerate form. The design needs an `Expired` probe or some other re-entry for an attached idle writer, and it must say what happens to the writer's remaining attempts.

3. **MEDIUM — D4: nothing says gate slots are free while the runner waits for the fixer.**
   - The fixer's own gate call needs slots, and the hub has `slots: 2` (template).
   - Counterexample: if the outer detached run keeps the slots across the wait, the fixer's gate waits for a slot its caller holds, and the fixer times out.
   - The record should state that the outer run releases every slot before it spawns the fixer and takes them again for the re-run. The row closure in D2 has the same need between the inner verify run and the fixer.

4. **LOW — D2: the deadline-overrun bound is too small.**
   - The record says a finalize evaluation can outrun the row's deadline by "at most one fixer timeout".
   - The wrapped closure runs the inner gate a second time after the fixer. For Darn-Write that is a full tiers 0–2 run of the `darn-write` key. The real bound is the fixer timeout plus one gate run.

5. **LOW — D2: "at most the entry's `attempts` plus one fixers per writer run" can be exceeded.**
   - `gateFails` and `gateSent` live in memory, and "after an attach a capped entry starts from zero" (`gate.go`, `PassOnCap` doc). The same reset applies to failure counts generally.
   - Each driver restart therefore resets the count, and more fixers can run. The bound holds per shuttle attach, not per row.

6. **LOW — D4, D6: the recursion guard checks only the `gate-fixer` role.**
   - D6 keeps webster's verify fixer under its own role (`verify-fixer`, `verifyFixerStrandRole`), and webster recovery strands also run card gates.
   - If a hub sets `fixer` on `webster-fix` or another key those strands call, the guard does not stop them from handing off.
   - Either every cold-fixer role should be guarded, or D6 should move the verify fixer onto `gate-fixer`.
   - The bound "a card gets at most the fork's self-fix cap plus one fixers" also ignores recovery strands re-running the same card's gate.

7. **LOW — D7: the lens review now runs before any validator.**
   - The Discussion writer's lens review now comes before validation. The fork boilerplate in the same stencil still tells forks the files "passed the self-check", which becomes false.
   - D7 should change that line too.

**Claims attacked**

- D2, dirty tree after the darn fixer passing the row unchecked: **held**. The verify gate fails on a dirty tree (`internal/loomshed/verifygate.go:133-134`), so the re-run catches it.
- D2, pending re-evaluation restarting from the first entry, which is why the closure blocks: **held** (from `evaluateGate`).
- D2, no `shuttleengine` change needed: **held** for the first evaluation; restart is broken (finding 2).
- D2, a passing closure spawns no fixer: **held**.
- D2, deadline overrun bound: **broken** (finding 4).
- D2, `attempts` + 1 fixers per run: **broken** across attach (finding 5).
- D3, template `card.fixer` surviving a hub's `card` tuning: **held**. `gates` is a per-name map merged name by name (`gateslot.ConfigPerNameMaps`), and its entries merge key by key (configengine doc).
- D3, fixer setting resolved through `Env.Setting` at build: **held**. It is wired in batten and loom.
- D4, recursion guard: **partly held** (finding 6).
- D4, the caller's re-calls join while the fixer works: **held** for the fork, **broken** for the fixer's own call (finding 1).
- D4, a cancelled detached run stops its fixer: not attacked further; plausible.
- D4, slots: **unstated** (finding 3).
- D1, attach after restart: **broken** for row callers (finding 2).
- D1, done/dead ordering, reused from `awaitColdFixer`: **held**.
- D5, edit scopes and leaving `Added after Discussion` entries alone: **held**.
- D6, unchanged trigger and ladder: **held**.
- D7, writer loop removal: **held**; stencil consistency (finding 7).

**Tool calls**

1. Grepped gateslot config for overlay and per-name map handling. Context lacked how `gates` merges.
2. Grepped gatecli for the detach run-key construction. Context lacked what identifies a detached run.
3. Grepped the verify gate's dirty handling and the shuttle attach-boundary lines. Context lacked whether the re-run catches a dirty tree and how an attach sets the boundary.
4. Grepped where `Env.Setting` is wired. Context lacked whether rows can read cross-module settings.
5. Read `gateslot` `ConfigPerNameMaps`, `detachKeyDir`, and the boundary and attach branches of `wait.go`. Context lacked their exact text.
6. Grepped the `detachKeyDir` caller and its `dir` argument. Context lacked what the key hashes.
