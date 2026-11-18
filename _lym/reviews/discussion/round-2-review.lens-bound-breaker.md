```yaml
lens: bound-breaker
outcome: reported
```

Task: attack every bound, guard and claim in the discussion pair (bound-breaker lens).

**Claims attacked**
- D1: `ReplaceStrand` persists the old strand's removal even when its add fails. **Held** (`internal/reedengine/replace.go`: the add is cut back and `reconcileApplyPersistLocked` still persists the removal).
- D1: a skip by `AddStrandUnless` counts as done, and neither side of a race between the two adds fails. **Held.** The skip matches the full name formed from `NameOverride`. A bootstrap `ReplaceStrand` on a GUID that orch already replaced logs a warning and returns nil, and D1's re-add or its skip covers the checklist.
- D1 bound: "only right after it took that strand's window, only for a run not done". **Held.** The re-add sits only on the successful-replace branch.
- D1: "A failure of the re-add … is a returned warning naming `lym loom start` as the way to bring the checklist back", and the failed-replace warning names it as well. **Broken.** See B1.
- D1 claim: `statusStrandAddSpec` already gives the checklist its own window when the runs strand came first. **Held** (`internal/loomcli/bootstrap.go:31-38`).
- D1/D2: the seed, status and sidecar belong to the session's own worktree, through `wireCLI`. **Held** (`internal/orchcli/cli.go:209`, `statusSidecarPath` takes a location).
- D2: the builder lives in a package both `internal/loomcli` and `internal/orchcli` already import, with no cycle. **Held.** `internal/statuspane` and `internal/loomengine` are both imported by both. Neither `reedengine`, `shedrun` nor `loomengine` imports `statuspane`.
- D2: with an unreadable recipe the group is empty and reed names the window by role. **Held** (`createOwnWindowPaneLocked` takes the `group == ""` branch to `strandLabel`).
- Constraint: reed refuses a second overview strand and a `Display.Window` group on an overview strand. **Held** (`strand.go:75`, `strand.go:405-409`).
- Constraint: `KeepViewer` never selects the new window. **Held** (`ownwindow.go:475-483`; only `Focus` overrides it, and the builder sets none).
- Constraint: `window_share_max` and the size-1 shared group. **Held** (`ownwindow.go:204-223`, `template_posix.yaml:9`).
- D3: window name is the group when the record is shared, else the role. **Held** for every reachable state. The only counterexample is a second live strand in a group of the recipe's name (`liveCount >= Size` sets `placeApart` while the record says shared). Nothing in this design creates that strand.
- Constraint: the overview takes reed's shared read lock and never refuses on an unreadable file. **Held.** `LoadState` errors on a corrupt file and returns nil on a missing one, so both read as "no window name". `AcquireReadLock` blocks (`internal/lock/lock.go:69-76`), but reed holds that lock only for the length of a write.
- Constraint: the finished test is `StateDone`, as in `runStatusUnfinished`. **Held** (`anchordeps.go:408-421`).
- Scope Out: "a later `lym loom start` or resume adds it back, as today". **Broken.** See B2.

**Findings**

- **B1 — MEDIUM, design, `_lym/discussion/decision-record.md#D1`**
  - Summary: D1's way forward to bring the checklist back, `lym loom start`, also puts a halted pawl run's driver back to work.
  - Counterexample: a pawl run sits at `blocked`, `failed`, `paused` or `awaiting` with a parked live driver, and D1's re-add (or the replace) fails. The operator follows the warning and runs `lym loom start`. Start sees `parkedLive` and calls `resumeParkedDriver`. With no driver left, it sees `mustSpawn` and spawns one. Either way the run is resumed or re-stepped while the operator only asked for the checklist. That is the hand-advanced stuck-pawl case D3's Rationale names.
  - D1 states no bound on this side effect.
  - Evidence: `internal/loomcli/start.go:330-366` (`mustSpawn`, `parkedLive` → `resumeParkedDriver`), `internal/loomcli/start.go:134-148` (`runHaltedAtHandBack`), `internal/loomcli/start.go:549-553` (status-strand step inside the same start).
  - Fix: name a way forward that restores the strand alone, or state in D1 that `lym loom start` also resumes a parked or dead driver and bound when that is acceptable.
- **B2 — LOW, consistency, `_lym/discussion/decision-record.md#Scope`**
  - Summary: the Scope Out bullet claims `lym loom resume` adds the closed checklist back, but resume never adds a strand. Only `lym loom start` restores it.
  - Evidence: `internal/loomcli/resume.go:236-238` ("It never spawns an LLM driver, adds or attaches a strand…"), and `ensureStatusStrand`'s only caller is `internal/loomcli/start.go:553`.
  - The support log's Rejected alternatives line "which a loom start or resume already restores" makes the same false claim.
  - Fix: drop "or resume" from the Scope Out bullet and the Rejected alternatives line.

**Tool calls**
1. Grepped `ensureStatusStrand` callers, start's driver/resume logic, `statuspane` importers in `loomengine`/`shedengine`/`agentname`, and `lock.AcquireReadLock`. The context lacked the path a `lym loom start` way forward reaches, cycle candidates one hop past the manifest, and the read lock's blocking semantics.
2. Read `internal/loomcli/start.go:130-160` and `326-380` and grepped resume's strand work. The context lacked what start does to a halted run's driver and whether resume adds a strand.
3. Read `internal/loomcli/resume.go:225-260` and `internal/loomcli/start.go:540-560`. The context lacked resume's contract and where start's status-strand step sits.
