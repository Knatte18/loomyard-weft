```yaml
lens: interactions
outcome: reported
```

Task: interactions lens over the decision pairs of `decision-record.md`. Two pairs break and five are partial.

**Pairs checked**
- D1 × D2, the graced lock and the long re-exec wait: held.
- D1 × D3, registering after the lock and the refused-start line with no run-process record: held.
- D1 × D3/AC, the worktree-gone exit writing its line to `dobby.log`: broken (F1).
- D1 × D4, the worktree-gone exit while a tidy is in flight: partial. The record is silent on settling the tidy (F5).
- D2 × D3, the wiring-failure path reusing the re-exec test: held.
- D2 × D4, `recordStart` writing start ticks: partial. A legacy record is never back-filled (F4).
- D2 × D5, the first re-exec onto the shipping binary with fds inherited from the old spawn: broken (F2).
- D2 × D6, the re-exec start heartbeat and no state change it did not see: held.
- D3 × D5, lines from processes holding no lock appending without rotating: held for `dobby.log`.
- D5 internal, the spawner rotating `dobby.out` while holding no lock: partial. It contradicts D5's own no-race rule (F6).
- D6 × PATTERN-daemon-wakeups, the heartbeat riding the per-minute idle ceiling: held.
- D7 × D8, the machine stamped on Land Done and the count by machine: held.
- D9 × D10, `Ensure` running before the check reads `adopt`: held.
- D9 × D10, the `lym gate test --gate adopt` way forward when the spawn fails: partial. It runs on an unreconciled key (F7).
- D10 × D11, the lock, the pid-and-ticks liveness test and the window before the first write: held, since the record states the bound.
- D10 × AC, the losing check leaving the log untouched while its fds are unstated: partial (F3).
- Every constraint against its decisions: held, apart from the items above.

**Findings**

**F1. MEDIUM, design.** Location: Decision 1 (worktree-gone exit), Decision 3, Decision 5, and the AC "A beam dobby whose worktree is removed … writes the exit line".
- Summary: the exit line cannot be written where Decision 5 sends it, because `dobby.log` lives inside the removed worktree.
- Detail: `LogPath` sits under `<root>/<anchorRel>/.lym/beam`, so a per-line open for append either fails, leaving no trace, or recreates a ghost `.lym/beam` tree at the removed worktree's path.
- The record names no other place for this one line, so the AC cannot be met as written.
- Evidence: `internal/hubgeom/dobbygeom.go:26,47-55` (`root = HubPath/worktree`, `beamDir = root/anchorRel/.lym/beam`, the lock and log paths in it).

**F2. MEDIUM, design.** Location: Decision 5 together with Decision 2 and the Goal.
- Summary: a dobby that lives through deploys by re-exec keeps the fd 1 and 2 the old binary's `SpawnDetached` opened on `dobby.log`, so raw output keeps reaching `dobby.log`.
- Detail: a re-exec keeps the process's fds. Until the next fresh spawn, which a dobby that survives every deploy may never get, raw output (logger Warn lines, panics, the `placeDobby` envelope) keeps going into `dobby.log`.
- Decision 5's per-line rename then moves the inherited fd's inode to `dobby.log.1`, and the next rotation deletes it.
- So "no raw process output reaches the file" and the size-bound rationale both fail on every existing hub.
- The record never says that a dobby whose stdout or stderr is still `dobby.log` repoints them to `dobby.out`.
- Evidence: the current `SpawnDetached` appends stdout and stderr to `dobby.log` (board entry, Part 2 "Seen"; `internal/dobbyengine/start.go`); the re-exec keeps the pid and the image's fds (`selfexec.go:77`, `syscall.Exec`, cited in the entry).

**F3. MEDIUM, design.** Location: Decision 10 and the ACs "A second check started while one runs exits 0 and leaves the first's record and log untouched" and "A check started over an existing check.json and suite.log writes a fresh record and an emptied log".
- Summary: the record says the spawner no longer truncates `suite.log` and the verb opens it, but never says where the detached child's own stdout and stderr go.
- Detail: if the spawner keeps today's wiring (child fds on `suite.log`), the losing check's exit envelope and its logger lines append to the running check's log, which breaks the AC.
- If they go to a null device, a panic of the check leaves no text anywhere, and Decision 11 shows only "ended without a verdict".
- Evidence: `internal/adoptcli/deps.go:198-227` (`command.Stdout = logFile`, `command.Stderr = logFile`).

**F4. LOW, design.** Location: Decision 2 together with Decision 4.
- Summary: a record holding this pid and no start ticks reads as a re-exec, and the re-exec path rewrites nothing, so the start-ticks field is never filled in for a legacy record.
- Detail: today `recordStart` returns early on a matching pid, and Decision 4 keeps the record untouched on a re-exec.
- Until a fresh start, a fresh dobby that reuses that pid after a reboot reads as a re-exec. It then takes the ten-second wait and posts a refused-re-exec notice it should not post. That is the case Decision 2's start ticks exist to exclude.
- Evidence: `internal/dobbyengine/loop.go` `recordStart` (`if found && record.PID == os.Getpid() { return nil }`).

**F5. LOW, design.** Location: Decision 1's new worktree-gone exit together with Decision 4.
- Summary: the new exit can end a dobby with `State.Tidy` in flight, and nothing settles or stops that tidy agent.
- Detail: Decision 4 covers the tidy only across a re-exec. A dobby that exits for good leaves the agent's strand open, which is the incident's own symptom.
- The record states no bound for this exit, nor for the owned-elsewhere one.
- Evidence: board entry, Part 1 "Seen" (the tidy strand stayed open until the orch removed it); `internal/dobbyengine/loop.go` `Deps.TidyShuttle`, `StopStrand`.

**F6. LOW, consistency.** Location: Decision 5, rotation of `dobby.out` at a spawn.
- Summary: the spawner rotates `dobby.out` while holding no lock, which contradicts Decision 5's own reason for letting only the lock holder rotate.
- Detail: two concurrent spawners, such as `lym beam start` and an `EnsureLive` spawn in Decision 2's exec gap, can each rename it, so one `.1` is lost.
- A spawn while a dobby runs renames the file that dobby holds as fd 1 and 2, so its later stderr lands in `.1`, which the next spawn's rotation replaces.
- Evidence: decision-record Decision 5 ("Only the lock holder rotates … so two renames never race") against the next sentence (the `SpawnDetached` rotation); Decision 2's Bound (a second dobby spawned in the exec gap).

**F7. LOW, design.** Location: Decision 9 together with Decision 10, the `lym adopt` warning when the check fails to spawn.
- Summary: the way forward `lym gate test --gate adopt` runs through `loadGate`, which reconciles nothing.
- Detail: when the spawn fails, no `Ensure` has run, so on a hub whose build stamp is stale the hand run reads the template's `adopt` (tier 0) instead of `finalize`'s tuned tiers (0, 1, 2). It then goes green on less than the check ran before.
- Decision 9 promises the migration lands before the check reads `adopt`, but only the hidden verb's path honours that promise.
- Evidence: `internal/gatecli/run.go:274-301` (`loadGate`, no reconcile); `internal/adoptcli/adopt.go:327` (the warning's way forward); `internal/gateslot/template.yaml` gates; `_board/_lym/config/gate.yaml:46`.

**Tool calls:** none. Every finding rests on files already in the inherited context.
