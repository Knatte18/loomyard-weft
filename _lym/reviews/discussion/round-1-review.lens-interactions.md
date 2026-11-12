```yaml
lens: interactions
outcome: reported
```

Lens: interactions. I checked each pair of decisions that act on the same state, ran the failure and restart paths with the other decision in force, and checked the acceptance criteria and constraints against the decisions they touch.

**Pairs checked**
- D1 × D11 (one directory read, one import from the bolt): held. D1's acceptance criterion makes an exception for the import.
- D1 × D4 (`testcover.Dir` names both the tracked maps and the untracked exit record): broken. See finding 4.
- D2 × D7 (inherited maps at older commits): held. The child's warp forks from a descendant of every map commit.
- D2 × D11 (pairs forked before the import): held. D2 says such pairs start with no maps.
- D3 × D7 (parent pass after a landing leaves unaffected maps at their old commit): held.
- D3 × D9 (a hand rebuild in a task stays out of the parent): held.
- D4 × D5 (uncommitted maps against preflight's clean check): held.
- D4 × D6 (only the owner writes, and other records commits sweep in the maps): held. Nothing else writes maps; `WriteMap` is called only from `internal/gatecli/testmappackage.go`.
- D4 × D7 (no write means nothing to commit): held.
- D4 × D10 (a killed call is committed only by the next call, and upkeep is off): held through D5. The leftover maps stay harmless to preflight.
- D4 × D9 (the dobby reads the prime's exit record, and a task pass writes the task's own): held.
- D6 × D7 (an owner whose stamp changes rebuilds every map, so each is written and committed): held.
- D6 × D11 (order of the owner refusal against the import and the removal): broken. See finding 3.
- D7 × D8 (unchanged maps keep their form, and D11 rewrites imported maps compact): held.
- D7 × D11 (import compacts the maps, then plans them): held.
- D7 × exit record `planned_head` (dobby start rule): held. The dobby reads only the exit record.
- D5 × D11 (half-imported prime directory while a call is killed): broken. See finding 1.
- D9 × D4 and the dobby (pass registration is shared across the hub): broken. See finding 2.
- D10 × D11 × D1 (who makes the first prime call): broken. See finding 5.
- D6 bound (no recorded owner) × D4 push: held. The conflict is stated.
- Acceptance criteria D1–D11 against their decisions: held, except that the D9 and D11 criteria do not reach findings 1, 2 and 5.
- Constraint `PATTERN-lym-files` (mirrored subpath) × D4: broken. See finding 4.

**Findings**

1. **MEDIUM, design**: D11's guard skips the rest of a killed import, and its removal then deletes the old maps it never imported.
   - **Location:** D11 (decision-record.md L100–L103).
   - **What happens:** a call killed partway through the import leaves some maps in the prime's `_lym/testcover/`. D5 exempts that directory, so preflight does not see the half state. The next call sees "the prime's directory holds" a map, so it imports nothing. Once those maps are committed, the removal deletes every `map.json` still left in `<bolt>/.lym/testcover/`. The maps that were never imported are lost and get the 44-minute rebuild D11 exists to avoid.
   - **Bound:** D11 states none for a killed import.
   - **Fix:** gate the import on a completion mark, or on "every old map is present in the prime", rather than on "no map".
   - **Evidence:** decision-record.md L100–L103 and L55–L57. Today the maps are written per package through `internal/gatecli/testmappackage.go:753` and `:772`.

2. **MEDIUM, design**: in a task, D9's pass shares the one hub-wide upkeep registration with the prime's dobby pass, and the record says nothing about it.
   - **Location:** D9 (decision-record.md L85–L88).
   - **What happens:** `registerUpkeepPass` registers under `runproc.HubOwner` and refuses while any other `RoleTestMap` process is live in the hub. The dobby's `testMapStep` treats any registered pass as live. So a task's hand pass, possibly a full 44-minute rebuild, blocks the prime's upkeep, and a prime pass refuses a task's hand rebuild.
   - **Also unstated:** a task's pass is registered under the hub, not the task's slug. `pairteardown`'s `runproc.End` for the slug therefore does not end it.
   - **Conflict with D9:** this contradicts D9's claim that each worktree runs "on that worktree's own maps" independently.
   - **Fix:** state whether the registration becomes per worktree, and which owner a task's pass registers under.
   - **Evidence:** `internal/gatecli/testmap.go:318-352`, `internal/dobbyengine/testmap.go:25-30` and `:54-60`, `internal/runproc/record.go:50-51`.

3. **LOW, design**: the record never says that D6's owner refusal comes before D11's import and removal.
   - **Location:** D6 and D11 (L61, L100–L101).
   - **Case:** on a non-owner machine, a prime call whose `_lym/testcover/` is still empty would import that machine's own bolt maps if the import ran first. That writes maps on a non-owner, the case D6 forbids.
   - **The opposite order has its own gap:** if the refusal comes first, a non-owner machine's own `<bolt>/.lym/testcover/` is never removed, because every call there refuses.
   - **Fix:** state the order, and say what happens to a non-owner's old directory.
   - **Evidence:** decision-record.md L61 and L100–L103.

4. **LOW, consistency**: D4 places the locks under the worktree's `.lym/testcover/`, but under D1 a map's lock no longer goes there.
   - **Location:** D4 (L50) and the Constraints line on `PATTERN-lym-files` (L108).
   - **Why:** under D1 a map sits under the tracked `_lym`. `lock.TrackedFileLockPath` then puts its lock in the checkout's never-tracked lock directory, mirroring the file's place under `_lym`, which is not `.lym/testcover/`. Today's lock sits beside a map in `.lym` only because that file is under no tracked lym directory.
   - **Also:** D1 has the one `testcover.Dir` name the tracked maps directory, while the exit record, `ReadExitRecord(dir)` and `WriteExitRecord(dir)`, uses that same dir today. D4 moves the record to the untracked side without naming the second accessor that needs.
   - **Fix:** say the map locks follow `lock.TrackedFileLockPath`, and that the exit record gets its own `.lym` accessor.
   - **Evidence:** `internal/lock/tracked.go:23-29`, `internal/testcover/coveragemap.go:24-28` and `:202-213`.

5. **MEDIUM, design**: with D10's default off, nothing makes D11's first prime call. Until someone does, D1 leaves every worktree with no maps.
   - **Location:** D10 × D11 × D1 (L92–L95, L100, L134).
   - **What happens:** D1 stops every reader from reading `<bolt>/.lym/testcover/`. D11 imports only on "a call in the prime". D10 turns the dobby's automatic pass off, and this hub's `_bolt/_lym/config/beam.yaml` does not hold the key, so no automatic call ever comes.
   - **Effect:** from the landing until someone runs `lym gate testmap` in the prime by hand, every gate reads `testcover.MissNoMap` and runs whole packages. Every pair forked meanwhile inherits no maps (D2). D11's rationale, saving the rebuild of maps that are current, holds only if someone makes that call, and the record names no one.
   - **Fix:** name who makes the first call, or have the readers fall back during the window.
   - **Evidence:** decision-record.md L92–L95, L100 and L134, and `internal/dobbyengine/testmap.go:51-53`.

**Tool calls**
- `grep internal/lock/tracked.go` for `TrackedFileLockPath`'s doc: needed to check D4's lock placement after D1. The manifest held only the call site in `coveragemap.go`.
