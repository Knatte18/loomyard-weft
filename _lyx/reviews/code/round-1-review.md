---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/battencli/overview.go:526-543
    origin: lens:code-correctness
    summary: "The overview draws a dobby section in a task pair's frame, though the acceptance criterion says a task pair's frame carries none."
    fix: "In overviewCmd, attach source.dobby only when acting is the prime's WorktreeName or a slug in hubgeom.DobbyStart(location).Beams(); otherwise leave source.dobby nil and log nothing more than a Debug line. Add a tier0 test (or an extracted helper with a row) that a task pair's name yields no dobby reader, and that the prime and a beam do."
  - id: F2
    severity: MEDIUM
    class: design
    location: internal/battencli/dobbyreads.go:175-187 and :303-306
    origin: handler
    summary: "Every overview poll takes a momentary write lock on the dobby lock and on each driven run's driver lock, so a probe can make a starting dobby or a hand-run lyx batten run refuse as another instance already live; the comments say the pane reads files only."
    fix: "Make the pane's probes non-acquiring: read liveness and pass-in-flight without taking the lock (for example probe the lock holder through a read-only check in internal/lock, or add a lock.IsHeld helper that does not create contention), keeping the dobby's own tick probe unchanged. If a try-acquire probe is kept, change the comments in dobbyreads.go and battencli/cli.go (it only reads ... without the records' locks) to name that the pane briefly takes the dobby and driver locks, and confirm the acquirers in sheddriver.DriveVerb and dobbyengine.Run retry or tolerate one refusal."
  - id: F3
    severity: MEDIUM
    class: design
    location: internal/battencli/dobbyreads.go:209-258
    origin: lens:code-tests
    summary: "The changed DobbyTasks behaviour has no test: open dependencies listed by slug, a pruned dependency skipped, and parent and owner read for every task with an empty status."
    fix: "Add a test over a boardkit fixture (tier1 if it needs a hub, otherwise tier0 over a temp board dir) that calls DobbyTasks with a released task depending on an open slug, a done slug and a slug absent from the board, and asserts PendingDependencies holds only the open one and Parent is set for that task."
  - id: F4
    severity: MEDIUM
    class: design
    location: internal/dobbyengine/starts_test.go:109-134
    origin: lens:code-tests
    summary: "The refusal text clearing is an acceptance criterion and the delete condition in updateStart gained a Refusal term, but no assertion shows Refusal cleared when a pass claims the task or its entry changes."
    fix: "In TestStartReady_RefusalsAndBackoff, after the entry-changed retry and after a first pass that leaves a status file, assert that h.stateRecord().Starts[a] has an empty Refusal (or no record)."
  - id: F5
    severity: LOW
    class: design
    location: internal/dobbyengine/view.go:418-430
    origin: lens:code-correctness
    summary: "BuildView drops every done run from the slot count, while the scheduler keeps a done run in d.runs while its pass is in flight, so a task can read ready while startReady holds it at max_runs."
    fix: "Count a done run whose DriverLockHeld is true in the slots computation (pass the filtered-for-listing runs and the slot-taking runs separately), keep it out of view.Runs, and add a BuildView row with a done run holding its driver lock that shows the next task AtMaxRuns."
  - id: F6
    severity: LOW
    class: design
    location: internal/dobbyengine/passes_test.go:200-223
    origin: lens:code-tests
    summary: "TestLog_OneLinePerStartAndPass covers the full and continue kinds only; the full-no-reopen name written by passKind.String is unasserted."
    fix: "Add a run whose pass is spawned as full-no-reopen to that test and expect the line 'pass <slug> full-no-reopen'."
  - id: F7
    severity: NIT
    class: consistency
    location: internal/battencli/overview.go:491-496
    origin: handler
    summary: "overviewCmd's doc comment and Short still say the pane belongs to the prime's runs strand, while the changed Long text and docs say the prime and each beam."
    fix: "Change the comment on overviewCmd and the Short text to name the runs strand of the prime and of each beam."
---

### [BLOCKING] Dobby section drawn in a task pair's frame

**Location:** internal/battencli/overview.go:526-543
**Issue:** `acting := location.WorktreeName` goes straight to `NewDobbyViewReader`, and `hubgeom.DobbyGeometry` fails only when the directory is missing, so a pane run in a task pair draws `dobby <pair>: not running; way forward: lyx beam start`. The decision record's acceptance criterion says a task pair's frame carries none, and its auto-mode assumption says a pane in any other worktree shows no section. `lyx orch start` ensures the pane only in the prime and in beams, so only a hand run reaches it, but the criterion is explicit and the way forward shown is wrong for a pair. No test covers the pair case.

### [MEDIUM] Overview polls take the dobby and driver locks

**Location:** internal/battencli/dobbyreads.go:175-187 and :303-306
**Issue:** `lockHeld` and `dobbyengine.Live` try-acquire the write lock to learn whether it is held. Before this diff only the dobby's own tick did that; the pane now repeats it every poll in the prime and in every beam, for each driven run. A hand `lyx batten run` that tries the driver lock while a pane probe holds it is refused with `a batten driver already drives this run` (`battenDriveRefusal`, `internal/battencli/drive.go`). The window is short but real. The doc comments also say the pane reads files only and without locks.

### [MEDIUM] DobbyTasks change has no test

**Location:** internal/battencli/dobbyreads.go:209-258
**Issue:** The decided change (open dependencies by slug, parent and owner read for every task with an empty status) is the one the board-read decision rests on. `status_integration_test.go` runs over an empty board, and `view_test.go` builds `TaskView` values by hand, so a break in `DobbyTasks` fails nothing.

### [MEDIUM] Refusal clearing is not asserted

**Location:** internal/dobbyengine/starts_test.go:109-134
**Issue:** The acceptance criterion says the refused start's text is cleared when a pass claims the task or its entry changes. The test asserts only that the text is written. Dropping the `Refusal == ""` term in `updateStart`, or the clear on a claim, would pass every test.

### [LOW] A done run with a pass in flight takes no slot in the view

**Location:** internal/dobbyengine/view.go:418-430
**Issue:** `drivePasses` keeps a done run in `d.runs` while a pass of it is in flight (`internal/dobbyengine/passes.go:149-152`), and `runsInFlight` counts it. `BuildView` removes all done runs before `slotsTaken`, so while a Land pass finishes a task can show ready that the scheduler holds back. The decision says the view can never explain a wait the scheduler does not have, or miss one it has.

### [LOW] full-no-reopen log name unasserted

**Location:** internal/dobbyengine/passes_test.go:200-223
**Issue:** `TestLog_OneLinePerStartAndPass` pins `full` and `continue`; the third `passKind.String` result is in the log contract in `doc.go` and has no assertion.

### [NIT] Stale prime-only wording in overviewCmd

**Location:** internal/battencli/overview.go:491-496
**Issue:** The `overviewCmd` comment and `Short` still read `the prime's runs strand`, which the changed `Long` text, `cli.go` and `docs/overview.md` no longer say.

## Rejected

- Lens code-tests finding 3 (no test that a task pair's frame carries no section): merged into F1, whose fix adds that test.
- Lens code-tests finding 5 (hub `Log` wiring to the writer unpinned): a one-line `Fprintln` wiring; the engine side is pinned by `TestLog_OneLinePerStartAndPass`, and a test of it would restate the wiring.
- Lens code-tests finding 6 (unasserted `owned by another machine` note and the unreadable line): static rendering strings in a function whose other cells are pinned; not a behaviour gap worth a row.
- Lens code-correctness F3, as a separate finding: kept as F2 with the handler's verification of the refusal path in `drive.go`.
- Lens code-affected-set raised no finding; its unchecked `orchcli/start.go` wording was checked by the handler and yielded only F7.
- The view showing a dead dobby's tasks as ready, and a task with an expired backoff reading as ready with its refusal text hidden: both follow the decided semantics (the header names the dead dobby; an expired backoff means the next tick retries).
- Focus items on the commit message, import allowlist, `lyx beam status` coverage, the stencil addition and the PATTERN.md need: checked, no defect.
