```yaml
lens: existing-paths
outcome: reported
```

Task: existing-paths lens. For each decision, trace the code that already writes, resets or reads the state the decision touches.

**Paths traced, per decision**

- D1, runproc registration against the processes that end a run. The only callers of `runproc.End`, `EndSparing` and `EndRole` are tender's cancel and owner-end paths, pair teardown, the fabric anchor path and the dobby role ends. Cancel stops the fixer: **survives**. `shedverbs/goto.go` neither ends processes nor stops strands, so goto stops nothing: **breaks** (F2).
- D1, finalize's strand and run-dir removal (`wait.go:1830-1841`). It removes the writer's shuttle run dir. A row fixer's record lives in the shed scratch dir, not the shuttle run dir: **survives**.
- D2, the arrival evaluation through `handleGatedBoundary`, with the writer idle at its boundary: **survives**.
- D2, the deadline finalize (`wait.go:416-420` → `finalize` → `evaluateGate(true)` at `wait.go:1784`). It runs whatever the writer is doing, including right after a re-prompt was sent. The wrapped closure would spawn a fixer then: **breaks** (F1).
- D2, the `Expired` probe after an attach (`wait.go:372-380`, `probeExpiry`). It is asked only off the boundary, and after an attach the writer is idle: **survives**.
- D2, the memoised verdict (`evaluateGate` memo, `wait.go:1609`). A finalize after a spent budget reuses the memo, so it spawns no extra fixer: **survives**.
- D2, Darn-Write's kept findings (`KeepGateFindings`, `entries_darnwrite.go:71`). They read `Findings`, which carries the fixer text: **survives**.
- D4, a superseding front (`startOrJoin` expires the old lease, `detachfront.go:594`; the old runner cancels at `detachrun.go:387`). D4 says the old runner stops its fixer and the next runner attaches to it or stops it. At worst the next runner waits on a fixer the old one is stopping, reads it dead and re-runs: **survives**.
- D4, `pruneOldRuns` (`detachfront.go:747`). It removes only runs older than 14 days whose runner is not live. The fixer record's report and reply paths are never pruned under a live fixer: **survives**.
- D4, `launchRunner` drops the strand name from the runner's environment (`detachfront.go:130`). D4 already moves the caller's role into the request: **survives**.
- D6, webster `VerifyFixers` with `RecordVerifyFixerTerminal` after a reset archived the state (`verifyfixer.go:197-207`). Unchanged by D6: **survives**.
- D8, the row record "keyed to the row visit" against `shedrun.ScratchDir`. ScratchDir is per run (`shedrun/paths.go:155`), with no visit segment. The record has to make up the visit key, and goto leaves the earlier visit's fixer running: **breaks** (F2).
- D3, the gate.yaml per-name map merge, as the Constraint states: **survives**.

**Findings**

F1. BLOCKING · design · D2 (Rationale, step 2, Bound "the deadline finalize after it runs the wrapped closure again")
The deadline finalize can spawn a fixer while the writer is mid-turn, so two agents edit one worktree at once.
The loop finalizes on the deadline whatever the writer's state (`internal/shuttleengine/wait.go:416-420`), and `finalize` runs `evaluateGate(true)` before it removes the writer's strand (`wait.go:1781-1784`, removal at `1830-1836`).
D2's own overrun bound counts this case: the evaluation fails, re-prompts the writer, and then the deadline finalize runs the wrapped closure again. So the writer has just been re-prompted and is working while the finalize spawns a fixer.
D2's rationale, "the writer sits idle at its turn boundary, so the fixer edits a worktree nobody else is touching", is false on this path. Go's re-run then judges a tree the writer is still changing, and on Darn-Write the writer may commit over or under the fixer.
The record never bounds or excludes this.
A fix would be: a final evaluation reached off the writer's idle boundary spawns no fixer, and only waits on a live one.

F2. MEDIUM · design · D1 ("ending the run's processes, by cancel, goto or reset, stops it") and D8 ("a goto, reset or row re-entry starts an empty set")
Goto stops no live fixer. With D8's visit keying, a fixer from the earlier visit becomes unreachable and keeps editing the worktree under the next row.
The only process-ending callers are `internal/tendercli/deps.go:113` and `deps.go:321`, `internal/pairteardown/teardown.go:603`, `internal/fabriccli/anchordeps.go:243`, and the dobby `EndRole` sites. `internal/shedverbs/goto.go` holds no runproc or strand reference.
Case: the driver dies while a row closure blocks on a live fixer, and the operator runs goto to another row. D8 then starts an empty record set, so no entry of the new visit waits on that fixer or attaches to it. The fixer edits on, unseen, while the next row runs.
D1's claim that goto stops the fixer has no existing path behind it.
A fix would be: goto and reset end the run's fixer role through runproc. Or D8 keeps the record reachable across a visit, so the next visit's wrapped entries wait on it or stop it.

F3. LOW · design · D8 ("The set is keyed to the row visit")
No row-visit identity exists to key the set by. `shedrun.ScratchDir` is per run, with no visit segment (`internal/shedrun/paths.go:155-157`).
The record treats visit keying as given. It names no source for the visit identity, so the claim that a row re-entry never reads an earlier visit's record rests on a mechanism the record leaves undefined.
A fix would be: name what identifies a visit, such as the run's step number in the steps records.

Tool calls:
1. grep of `runproc.End`/`EndSparing`/`EndRole` callers: the manifest held no reader of the run's processes, which D1's cancel/goto/reset claim needs.
2. `shedrun.ScratchDir` definition: the manifest named only `RunScratchDir`'s wiring, not its path shape, which D8's visit keying needs.
3. grep of `StrandNameEnv` in shuttleengine: to check how a strand spawned from a process with no strand name gets its parent. It came back inconclusive and yields no finding.
4. `tendercli/deps.go:300-330`: to identify which tender path calls `runproc.End` with the slug.
5. `tendercli/deps.go:95-116` plus a grep for goto files: to identify the cancel/owner-end caller and find the goto verb.
6. grep of `goto.go` for runproc and strand stops: to check whether goto ends a run's processes.
