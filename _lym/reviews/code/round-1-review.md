---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/dobbyengine/starts.go:89
    summary: the hold branch runs before the ownership, dependency and pause checks, so a task another machine owns, or one with open dependencies or a covering pause, gets a refusal record and a creel notice from this dobby
    fix: in startReady, apply the hold branch only when the task is not owned elsewhere (task.OwnedElsewhere false); optionally also when it has no pending dependencies, so the notice comes from the dobby that would start the task.
  - id: F2
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/beamcli/cli.go:73
    summary: the lym beam status help text lists what a task waits on and omits the new start batten refuses reason and the last refused start on a ready task
    fix: in the status help text at internal/beamcli/cli.go:73-75 add "a start batten refuses" to the list of waits and say a ready task shows its last refused start when one is recorded, matching battencli/cli.go:27.
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/dobbyengine/view_test.go:32
    summary: no row pins that a held task takes no max_runs slot in BuildView
    fix: add a TestBuildView row with MaxRuns 1 and a task with StartRefusal listed before a ready task, expecting the ready task to carry no AtMaxRuns.
---

### [LOW] Hold notices a task that another dobby or condition owns

**Location:** internal/dobbyengine/starts.go:89
**Issue:** `startReady` checks `task.StartRefusal` right after `PauseCovers` and before `readyToStart`, and `heldStart` ignores `OwnedElsewhere`, `PendingDependencies` and `covered`. A done record that arrived from another machine, the incident's own origin, on a task that machine owns makes this dobby post a notice it cannot act on. Before the diff such a task was skipped silently.

### [LOW] beam status help text not updated

**Location:** internal/beamcli/cli.go:73
**Issue:** The help text enumerates what a waiting task shows and still lists the old reasons; the package doc in `battencli/cli.go` was updated, so the two disagree.

### [LOW] No test for the slot a held task leaves free

**Location:** internal/dobbyengine/view_test.go:32
**Issue:** `heldByEntry` now includes `StartRefusal`, which keeps a held task from consuming a `max_runs` slot in `BuildView`. The `finished` row has one task and spare slots, so a regression would not fail it.

## Rejected

- Warn on every tick for an undecodable status file (code-correctness F2): the run listing at `dobbyreads.go:96` logs the same way, and the decision record accepts the undecodable case.
- No test for the hold applying only to a released task (code-tests): `DobbyTasks` fills `StartRefusal` only for a released task, so a non-released carrier is not reachable.
- Untested hold-plus-pause order (code-tests NIT): covered by F1, which decides the order.
- Held task gets no notice while at `max_runs` (code-correctness remark): the refused-start path behaves the same.
