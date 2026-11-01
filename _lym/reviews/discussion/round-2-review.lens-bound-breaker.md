```yaml
lens: bound-breaker
outcome: reported
```

Bound-breaker lens: attack every bound, guard, ordering promise and code claim in the decision record.

**Claims attacked**
- D1 join liveness, "the process file names a live process with the recorded start time": **broken on Windows** (F1).
- D1 claim wait, "a late claim is refused": **broken** (F5).
- D1 join over HEAD's tree and the dirty path set, through `verifytree.DirtyPaths`: held. It is `git status --porcelain`, `.lym/` is gitignored, and the runner's own files never change the set.
- D1 recorded 75 replayed as exit 1, and the runner never answers a slot-busy 75 under `--wait` semantics: held.
- D1 direct call of the runner verb: held as far as the stated bound goes (lease, claimed process file, lease end).
- D1 second front after delivery starts afresh: held as stated.
- D2 "a nested call under a verify or card-verify line runs in-process": held. A card-verify `--wait` line is in-process by `--wait`, and drift's shell sets `HolderEnv`, so it inherits a slot (`internal/driftengine/steprun.go:103`).
- Constraint "every `verifyrun.Run` caller removes the strand", batten's two sites passing a nil env: held (`internal/battencli/land.go:155,181`). The other `ShellFor` users are drift (inherited slot), gate tiers and build (strand stripped) and desktophook (window command). None detaches a Go caller.
- D3 "the lease is the latest deadline; the supersede expiry is the one write that moves it earlier": **broken** (F4).
- D3 lease written before the spawn: held as stated.
- D3 "a runner killed without its exit file frees its slots with its OS file locks": **broken for a D6 runner** (F3). It holds for a `lym gate test` runner, whose tiers are `StartTied` with `Pdeathsig` (`internal/proc/tied_linux.go:18-19`).
- D3 SIGTERM through `runproc.End`: held through the fallback. The grace is 5 s, then SIGKILL (`internal/runproc/end.go:20`). An exit file that is not written in time takes D1's dead-process supersede path.
- D3 "pair teardown and a session end that ends the slug's processes end it": **broken as a claim about existing code** (F7).
- D3 abandoned-runner bound (latest deadline plus grace): held, plus at most one second of lease read and the kill time.
- D4 validation, "each cap below its role's lifetime" as the keep-warm guarantee: **broken** (F2).
- D4 long cap of 55m inside the one-hour Bash ceiling: held, even with the 30 s claim wait and the lock wait added.
- D5 "a record carries no mark of a cancelled call": held (`internal/gateslot/runlog.go:38-73`).
- D5 "`execution.report` writes the tier the cancel cut": held (`internal/gatecli/run.go:479-496`).
- D5 "a build step's record has a zero `WaitStart`": held (`internal/gatecli/build.go:63-66`).
- D5 "wall time includes the slot wait the front blocks through": **partly broken** (F6).
- D6 "a cancelled line's group gets SIGTERM, then SIGKILL": held (`internal/verifyrun/run.go:39,83`).
- D6 "a `--wait` gate child cancels on SIGTERM and frees its slots": held while the runner lives. Its tiers are tied to it.

**Findings**

F1 — BLOCKING, design. D1 join rule and D3 Windows bound (`decision-record.md:48,51,106`).
The join's liveness test cannot work on Windows.
- `proc.StartTime` answers `"", false` there (`internal/proc/proc_windows.go:54-55`), and `runproc` signals nothing (`internal/runproc/end.go:57`).
- So the runner records no start time, and a front can never confirm "a live process with the recorded start time".
- If no confirmation means no join, every re-run supersedes the live runner and starts afresh. A gate longer than the cap then never returns a verdict, and the agent re-runs forever.
- If the empty time counts as a match, a reused pid passes as live.
- D3 covers Windows only for how the runner ends, not for the join.

F2 — MEDIUM, design. D4 validation against D1's waits (`decision-record.md:49,57,111`).
"Each cap below its role's lifetime" does not bound the time the agent blocks.
- A starting front first waits up to 30 s for the runner's claim, then waits its cap on the exit file.
- A front that joins can first block up to 30 s on the key directory's lock while another front holds it for its own claim wait.
- Counterexample: `keep_warm_cap_roles: {implementer: 290s}` on a 5-minute lifetime passes validation. A starting call then blocks about 320 s and returns to a cold cache.
- The declared `PATTERN-keep-warm-cap` makes the resolved cap the bound of the agent's wait. Here the claim wait and lock wait sit outside it.

F3 — MEDIUM, design. D3 kill bound and D6 card-verify runner (`decision-record.md:95,148`).
A card-verify runner killed by SIGKILL, an OOM kill or a crash leaves its line running and holding slots.
- `verifyrun.Run` starts the line's shell with `ConfigureGroupTerminate` (`internal/verifyrun/run.go:83`). That is a new process group with no parent-death signal; only `StartTied` sets `Pdeathsig` (`internal/proc/tied_linux.go:19`).
- So when the runner dies, a `--wait` gate child goes on to its end: it holds its own slots and runs its tiers, outside any lease.
- A non-gate line's group goes on under the runner's slot. Here the slot does free with the runner's lock, so the line runs on without one.
- The next front finds the runner gone and starts a fresh runner. Its line queues behind the orphan or runs beside it.
- D3's "frees its slots with its OS file locks" is false for this path.
- `runproc.End` does reach descendants, so only the unsignalled kill path breaks.

F4 — LOW, consistency. D3 lease monotonicity (`decision-record.md:90` against `:50`).
D3 says the supersede expiry is "the one write that moves it earlier". D1's start failure also expires the lease when the claim times out.

F5 — LOW, design. D1 "a late claim is refused" (`decision-record.md:50,55`).
The runner checks the lease, then creates its process file. Nothing orders that against the front's expiry.
- Counterexample: the runner reads a valid lease at 29.9 s. The front expires the lease at 30 s, exits 1 and releases the lock. The runner's exclusive create succeeds at 30.1 s.
- The claim is not refused. The run ends only at its next lease read, about one second later, after it may already have started its slot wait.
- The guarantee needs the claim made under the key lock, or a lease re-check after the claim.

F6 — LOW, design. D5 wall time (`decision-record.md:133`).
The span starts at the first record's `WaitStart`, so it leaves out the pre-tier phase.
- That phase is config load, scope computation and the guard scan, bounded by `preTierCeiling` (`internal/gatecli/run.go:53-90`).
- It runs before the first tier's `waitStart` is taken (`internal/gatecli/run.go:460`), and the front blocks through it.
- The median therefore runs short for keys with a long scope or guard scan.

F7 — LOW, consistency. D3 registration claim (`decision-record.md:97`).
"A session end that ends the slug's processes" names a path that does not exist.
- The code's enders are `pairteardown` (`internal/pairteardown/teardown.go:603`), `lym tender`'s `End` (`internal/tendercli/deps.go:113,321`), the dobby `EndRole` calls, and `lym fabric anchor`'s `EndDobbies`.
- `EndDobbies` is an `EndSparing` that ends every role it does not spare (`internal/fabriccli/anchordeps.go:243`).
- So an anchor during a gate run also ends the runner, which the record never mentions. It costs one gate run; the abandoned mark keeps it from becoming a verdict.

**Tool calls**
1. `internal/proc` Windows `StartTime`/`IsAlive` and the `Pdeathsig`/`Setsid`/`ConfigureGroupTerminate` sites: the manifest held the Linux tie only, not Windows liveness or verifyrun's group setup.
2. `verifytree.DirtyPaths` and the `.gitignore` entries for `.lym`: the join's dirty-set source was not in context.
3. `verifyrun.Run` and `ShellFor` callers: to look for an operator-command spawner the strand constraint misses.
4. `internal/driftengine/steprun.go` spawn and environment: the one `ShellFor` user outside gatecli whose environment was unknown.
5. `runproc.End`/`EndSparing`/`EndRole` callers: to test D3's claim about which paths end the runner.
6. The `anchordeps.go` `EndDobbies` and `teardown.go` spare context: to confirm what the anchor and teardown ends reach.
