---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D4
    summary: D4's premise that the watcher's binary is the one the orch pane's prelude exports is false on the StartSpawnWatcher path and on a reed resume relaunch of the orch strand
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D4 Bound
    summary: D4's missing-binary bound says the render fails and the watcher goes idle as a missing stencil does, but startAutoReload returns a render failure as a tick error, and 30 in a row end the watcher
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D9
    summary: D9 forbids a group Long restating its subcommands, but the root Long lists every module and TestCLITree_EveryCommand requires it; the record does not say whether the root is exempt
---

### [MEDIUM] D4 names one binary for the watcher and the orch pane, but they can differ

**Location:** `_lyx/discussion/decision-record.md`, D4, the sentences "The index describes the binary the orch calls: the `lyx` at the absolute path of the binary that ran `lyx orch start` or `--adopt`, which reed's pane prelude puts first on the orch pane's `PATH`" and "`<path>` is the absolute path `spawnWatcherProcess` already starts the watcher from".

**Issue:** The watcher's path and the pane's `LYX_BIN` coincide only when one `lyx orch start` both launches the strand and spawns the watcher, the `StartRelaunch` branch in `internal/orchcli/start.go`.
Two other paths break the premise:

- `orchengine.DecideStart` returns `StartSpawnWatcher` when the strand is live and no watcher holds it (`internal/orchengine/launch.go`).
  `lyx orch start` then calls `spawnWatcher` without relaunching the strand, so the new watcher runs from `os.Executable()` of whichever `lyx` the operator typed now (`internal/orchcli/spawn.go`), while the pane keeps the `LYX_BIN` of the binary that launched it earlier, for example `.dev-bin` against the installed one.
- The pane prelude is composed from `executablePath()` of whatever process calls `launchStrandLocked` (`internal/reedengine/panebin.go`, `composePaneLaunchLine`).
  A reed resume relaunches strands from the calling process (`internal/reedengine/lifecycle.go`, the `resume strand` loop), so a revived orch pane takes the `LYX_BIN` of the `lyx` that ran that reed command, and the watcher's recorded path does not follow it.

In both cases the watcher's render hands the orch an index of a binary it does not call, the defect round 2 set out to remove.
The focus question, whether the record names one binary for the watcher, start, adopt and `resume-context`, therefore resolves to: it names one, on a premise that holds for one start branch only.
`resume-context` runs inside the pane and is unaffected.

**Fix:** Either give the watcher's index source the pane's own binary (for example a path recorded wherever the orch strand is launched or relaunched, read at each render), or keep the recorded watcher path and state the mismatch under `## Open risks` with the paths that cause it, and reword D4's premise so it claims equality only for the `StartRelaunch` branch.

### [MEDIUM] D4's missing-binary bound does not match the auto-reload render site

**Location:** `_lyx/discussion/decision-record.md`, D4, "Bound: ... when the path no longer exists or the run fails, the render fails and the watcher goes idle with the reason, as a missing stencil does."

**Issue:** Of the three watcher render sites, `startClearing` and `reloadAfterCompaction` turn a `RenderRoleFile` error into `w.toIdle` with the reason (`internal/orchengine/watcher.go`).
`startAutoReload` returns the error unchanged.
The watch loop counts a returned error as a failed tick and records an exit after `maxConsecutiveTickErrors` (30) in a row (`internal/orchengine/watchloop.go`).
With D4's source, a missing recorded path at an auto-compaction reload means one failed `help index` spawn per tick, then the watcher exits; it does not go idle.
The bound the record states is the one a plan writer would test against, and it is false for this site.
Changing `startAutoReload`'s error handling may also fall under Scope's "Out: ... the watcher's reload steps", which the record does not reconcile.

**Fix:** State what bounds the render at `startAutoReload`: either D4 brings that site in line with the other two (and Scope's out-list says that this one change is in), or the bound names the actual behaviour (a tick error per poll, the watcher exits after the consecutive-error limit) and the acceptance criterion for a missing path covers the auto-reload site.

### [LOW] D9's group-Long rule and the root's module list disagree

**Location:** `_lyx/discussion/decision-record.md`, D9, "A group's `Long` does not restate its subcommands in prose, since cobra lists them."

**Issue:** The root command's `Long` in `cmd/lyx/main.go` carries "Available modules: board, config, ...", and `TestCLITree_EveryCommand`'s subtest "root.Long names every mounted module" fails unless every visible module is named there.
The root is a group, so D9 read literally removes the list and breaks that test; D8 already names `lyx` among the group `Short` texts it rewrites, so the record does touch the root.

**Fix:** Add to D9 either that the root's module list stays, with the reason (the existing test pins it), or that the list and its subtest go.

## Focus departures

None of the three directives was departed from.
Directive 1 (`help` against the tree walk): D1 and the acceptance criteria state a conforming behaviour and a named exemption with its reason: `help` is exempt from the bare-group check alone because bare `lyx help` prints the root's help, `lyx help bogus` passes the unknown-subcommand check, and both `help` and `index` carry `operator` and are reached by the audience and form tests once the walker's `help` skip goes.
Verdict: not a defect.
Directive 2 (which binary): see F1 and F2.
Directive 3 (`lyx help <group> <unknown-word>`): D1 resolves the topic word by word and refuses the first unknown word with `unknown subcommand`, naming the built-in `Find` fallback it replaces, and an acceptance criterion covers `lyx help board bogus` beside `lyx help no-such-topic`.
Verdict: not a defect.
