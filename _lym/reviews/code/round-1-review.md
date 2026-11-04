---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    origin: lens:code-affected-set
    location: internal/reedengine/lifecycle.go:467
    summary: PATTERN-reap-children, added by this diff, is violated by two child starts that never wait, the tmux server start in reedengine and vscode Launch
    fix: In internal/reedengine/lifecycle.go spawnSession, call proc.Reap(cmd) after the successful cmd.Start() and before the Info log. In internal/vscode/launch_linux.go Launch and internal/vscode/launch_windows.go, call proc.Reap(cmd) after the successful cmd.Start(), importing internal/proc (check the vscode package's import set first). If a site cannot import proc, narrow the PATTERN-reap-children text in PATTERN.md to name the exemption instead.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/beamcli/cli.go:386
    summary: The startup reap that fixes the reported zombie pile has no test, since the only runDobbyWith test passes a no-op reap and no test touches orch watch's reap
    fix: In internal/beamcli add a test over runDobbyWith that passes a recording reap and asserts it runs once after a successful placement and not at all when placeDobby refuses; extend TestRunDobbyWith_TracesEachExitBeforeRun or add a table row there rather than a new test where its setup allows. For orch watch, give the RunE reap a seam (a func field on orchCLI defaulting to the proc call) and assert in the existing watch test that it is called before LoadState.
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/runproc/record.go:187
    summary: List deletes the record of a zombie leader, which End keeps listed so its group signal reaches members that still run, so an earlier List from tender status or pair teardown orphans those members
    fix: "In internal/runproc/record.go list, split dead from zombie. Remove the record file and lock only when the pid is gone or carries another start time, and leave a zombie record on disk while omitting it from the returned slice. The record is then pruned as dead once the zombie is reaped. Add a case to TestRegisterAndList that the zombie record's file survives the list."
  - id: F4
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/fabricengine/spawn.go:66
    summary: Doc comments in fabricengine/spawn.go and adoptcli/deps.go still say the child is never waited for, though the diff made both call proc.Reap
    fix: In internal/fabricengine/spawn.go:66 change the sentence "started but never Waited" to say the child is reaped in the background through proc.Reap. In internal/adoptcli/deps.go:222 change "started and never waited for" the same way.
  - id: F5
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/landing/summary.md:33
    summary: The change description claims the reap runs in the cobra RunE and not in runDobbyWith, and that the end-sequence fake now reports a zombie as not running, and the diff bears out neither
    fix: In _lym/landing/summary.md correct the design decision to say the dobby's reap is the reap seam runDobbyWith calls after placeDobby, and delete the Tests sentence about the end-sequence fake, whose net change in end_test.go is nil.
  - id: F6
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/runproc/record.go:121
    summary: The production wiring of the zombie reading, proc.IsRunning in List and Owners and proc.IsAlive in productionEnder, is pinned only through fake seams, so swapping either fails no test
    fix: Add a Linux-only case to the runproc tests that starts a short child, lets it exit unreaped, registers it, and asserts List returns nothing while productionEnder's lister still lists it; place it in record_test.go or a record_linux_test.go.
  - id: F7
    severity: NIT
    class: consistency
    origin: lens:code-correctness
    location: internal/runproc/end.go:141
    summary: The lister field isRunning is bound to e.isAlive, a name that reads the opposite of what the line does
    fix: "In internal/runproc/end.go end, bind the field through a local named for the intent, such as keepsZombie := e.isAlive, and pass that local to the lister."
---

### [BLOCKING] A new PATTERN entry is violated by two child starts

**Location:** internal/reedengine/lifecycle.go:467, internal/vscode/launch_linux.go:20, internal/vscode/launch_windows.go:22
**Issue:** `PATTERN-reap-children` says a child's starter waits on it, by `cmd.Wait` or `proc.Reap`, with no short-lived exemption. The tmux server start in `spawnSession` and the `code` launch call `Start` and drop the command. Whether dobby or the watcher reaches them was not traced, but a long-lived reed process that respawns the session would leave the zombie the Goal describes. The repository treats any violation of a listed constraint as BLOCKING.

### [MEDIUM] The startup reap has no test

**Location:** internal/beamcli/cli.go:386, internal/orchcli/watch.go:98
**Issue:** The reap's placement is the behaviour that ends the zombie pile, and it must run before the process starts a child. The only `runDobbyWith` test passes `func() {}`, and no test covers `orch watch`'s call. Deleting either call, or moving it after a spawn, fails nothing.

### [LOW] List removes a zombie leader's record that End needs

**Location:** internal/runproc/record.go:187
**Issue:** `End` keeps a zombie leader's record so the group signal reaches members still running (`end.go:140`). `List`, called by tender status, the fabriccli listings and pair teardown, now deletes that record first. A later `End` finds nothing and the live members stay. Rare, since it needs a leader that exited unreaped while its group runs, but the result is a silent orphan.

### [LOW] Doc comments still say the child is never waited for

**Location:** internal/fabricengine/spawn.go:66, internal/adoptcli/deps.go:222
**Issue:** Both functions now call `proc.Reap`; the comments above them say the opposite.

### [LOW] The change description claims what the diff does not do

**Location:** _lym/landing/summary.md:33
**Issue:** The summary says both callers run the reap in the cobra `RunE` and not in `runDobbyWith`; the diff wires it through `runDobbyWith`'s `reap` seam. It also says the end-sequence fake now reports a zombie as not running; the net diff leaves `end_test.go` unchanged and the fake's `isAlive` still counts a zombie as alive.

### [LOW] The zombie reading is pinned only through fakes

**Location:** internal/runproc/record.go:121
**Issue:** `List` and `Owners` use `proc.IsRunning`, `productionEnder` keeps `proc.IsAlive`. Both are reached only through hand-built seams, so swapping either function breaks no test.

### [NIT] A field named isRunning is bound to an alive check

**Location:** internal/runproc/end.go:141
**Issue:** The comment explains the choice, but the line reads as the opposite of its intent.

## Rejected

- Code-correctness F2, the reap-before-child ordering is a caller convention: both call sites run before any child start (`placeDobby` runs git in-process; the watch `RunE` is preceded only by `clihelp.ShouldAbort`), and `reap.go` documents the contract.
- Code-correctness F3, the reed tmux start: folded into F1.
- Code-tests F3, no test pins each `proc.Reap` call site: the entry has no `(test)` marker, the sites are guarded by `testing.Testing()`, and `TestReap` pins the helper.
- Code-affected-set finding 4, the stale comment in `loomcli/start.go:574`: the comment predates the diff and the file is untouched.
- Code-affected-set finding 5, hand-rolled `go func() { _ = cmd.Wait() }()` sites: they satisfy the entry's `cmd.Wait` wording.
