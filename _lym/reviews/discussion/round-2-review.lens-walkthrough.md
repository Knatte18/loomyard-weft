```yaml
lens: walkthrough
outcome: reported
```

Task: the walkthrough lens. I ran each decision end to end with one concrete case and checked the end state against the decision's own purpose.

**Walkthrough list**

- **D1**, agent strand, `lym gate test --gate x` with tiers lasting 12 min. Steps: lock, write the lease, spawn, claim wait, `AwaitAny` up to the cap, then 75 with the phase file. On re-run, the front joins over an unchanged tree and dirty set (the worktree reads clean while `_lym` writes happen), replays the result, marks it delivered, and a third call starts fresh. **Purpose not met in one case:** a front can block for longer than its cap (finding 1).
- **D2**, implementer fork running `lym gate card-verify 4` whose line is `lym gate test --wait …`. The front is capped. `cardverify.Runner` strips the strand, so the `--wait` child runs in-process. **Purpose met.**
- **D3**, front with a 250 s cap abandoned after one 75. The lease passes at deadline plus 120 s. The runner cancels its call, and `runTest` halts through `report` (`e.ctx.Err()`), exiting with `haltExitCode`. The runner writes exit 1 and the abandoned mark itself. A SIGKILLed runner's tiers die through `Pdeathsig` (`proc/tied_linux.go:19`). **Purpose met.**
- **D4**, strand `implementer-2`. The numbered suffix is dropped and the template gives 250 s. The validation checks the cap against the lifetime alone (finding 1). **Purpose met** for the template cap; not for a valid cap near the lifetime.
- **D5**, `--gate x` with tiers 1 and 2, the second opting into the build, cancelled during the build step. The records are written through `build.go`'s direct `writeRecord` and through `reportBlocked`. **Purpose partly missed** (finding 2). The wall time also leaves out pre-tier work (finding 3).
- **D6**, a `--wait` gate line, lease passes. `verifyrun` sends SIGTERM to the group, the child gate's signal catch kills its tiers, and `execute` returns "failed to start: …context canceled". The runner records exit 1 with the mark. A non-gate line cancelled in `Acquire` gives "could not acquire a gate slot", exit 1 with the mark. **Purpose met.**
- **D7**, the implementer's card gate rendered without `--wait`. The agent's card `**Verify:**` goes through `lym gate card-verify` (`webster-body-implementer.md:75`), so the line's own `--wait` never blocks the agent directly. **Purpose met.**
- **D8**, the driver's `lym shed step` is unchanged. **Purpose met.**
- **D9**, both gate-rule steers in `shellguard/rows.go:161,176` carry "no gate slot is free yet", which D9 replaces. **Purpose met.**

**Findings**

1. **MEDIUM, design. D1 front wait vs D4 cap validation** (`decision-record.md:49`, `:57`, `:111`).
   - A starting front holds the key lock through the claim wait, up to 30 s. Only after that does it wait on the exit file for its full cap.
   - A joining front first waits for that lock, which another front can hold for up to 30 s.
   - D4 validates the cap only as "below the lifetime its role resolves to". So a valid cap of 290 s on a 5-minute lifetime makes a start call block about 320 s, and the agent comes back to a cold cache. That misses D1's purpose.
   - If the lease deadline is computed before the claim wait, the front answers up to 30 s after its own deadline, which cuts the 120 s grace to 90 s.
   - Fix: count the cap from the front's start, so the lock and claim waits fall inside it, or have D4 refuse a cap whose cap plus 30 s reaches the lifetime.

2. **LOW, design. D5 `Abandoned` only covers records written through `report`** (`decision-record.md:132`; `internal/gatecli/build.go:63-70`, `:83-90`; `internal/gatecli/run.go:483-485`).
   - The build step's record goes through `writeRecord` directly, and a blocked tier's record is built in `reportBlocked`. Neither passes through `report`'s record path, which is where D5 sets `Abandoned`.
   - Case: a cancel during the build of the last listed tier's slots. The build command fails, so `reportBlocked` writes that tier's record, and every listed tier then has a record with none marked.
   - The cancelled call then counts as a full call with a truncated wall time.
   - Fix: set `Abandoned` in `execution.writeRecord`, which every record of a call passes through.

3. **NIT, consistency. D5 wall time leaves out the front's pre-tier block** (`decision-record.md:133`; `internal/gatecli/run.go:36-110`).
   - The records start at the first tier's `WaitStart` or the build's `Start`. Config load, scope computation and the guard scan run before that, under `preTierCeiling`, and the front blocks through them.
   - So "includes the slot wait the front blocks through" understates what the front blocks through, and the median runs low.

Tool calls:
1. `verifytree.DirtyPaths`: the D1 join compares dirty sets and the manifest lacked how they are read.
2. `git status --porcelain` in the worktree and `_lym` log: to check whether lym's own writes change the dirty set during a wait.
3. Grep of the stencils for `card-verify`: to walk how the implementer runs a `**Verify:**` line (D6/D7).
4. Grep of `runproc` End/EndSparing/EndRole callers: D3 says a session end and teardown end the runner.
5. `fabriccli/anchordeps.go` EndSparing caller and the teardown spare list: one hop past item 4, for the D3 end path.
