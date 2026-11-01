---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/fronttask/hub.go:114
    summary: startAction, which spawns the land action and logs the spawn, has no test; it returns early under go test and TestSettle reaches it only through a fake RunAction
    fix: In internal/fronttask, make the executable lookup and process start a seam of startAction (or test the actions row's args function and the logged spawn directly), and add a fronttask test that asserts the spawned arguments equal "adopt check <subject>", that the spawn is logged, and that an action key outside the table is refused.
  - id: F2
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/adoptcli/fixloop.go:failedEnd (fronttask.File error return)
    summary: a File failure at the record write (entry created and released, try count already raised) is reported as "not filed" with a re-run advice, though the released task will start
    fix: In fronttask.File treat a failed record write as done after logging it (Settle records an open front entry with no record), or return a typed error that failedEnd words as "filed, record missing"; add a TestFailedEnd or TestFile row for a failing record write.
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/loomshed/darnwrite.go:149
    summary: the condition-stop branch sets the halt reason to the stop report alone, so an empty report halts the run with no reason and no way forward
    fix: In the conditionStopped branch of darnWrite.Call, when the trimmed report is empty set the reason to a fixed message that says the writer stopped on the task body's condition and wrote no report, naming the stop marker path; add a TestDarnWrite_Call row for the empty report.
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/adoptcli/green.go:resumeHaltedChildren
    summary: ChildStatus is read for every non-done board task before the subject filter, so a failed status read of another subject's child adds a way-forward line to this subject's notice
    fix: In resumeHaltedChildren call deps.ParentOf and skip the children of other subjects before deps.ChildStatus; adjust the TestGreenEnd row that reads the status of another subject's child.
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/fronttask/hub.go:74
    summary: HubSettleDeps and its owner read (the owner-only rule on a real board) have no test; only the OwnedHere fake in TestSettle is covered
    fix: Add a hub_integration_test.go test that builds HubSettleDeps over a hub whose board names another machine as the subject's owner and asserts OwnedHere is false, and true when this machine owns it.
  - id: F6
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: docs/overview.md:458
    summary: the overview describes neither internal/fronttask nor dobby's start-ahead and on_land behaviour for a front task
    fix: In the dobby and beam entry of docs/overview.md add a short paragraph naming internal/fronttask, that an open front task starts first and past max_runs while no other task of its worktree starts, and that its on_land action runs from a closed table on the subject's owner only.
---

### [MEDIUM] startAction has no test

**Location:** internal/fronttask/hub.go:114
**Issue:** The spawn of a land action (argument vector, detached start, spawn log required by `PATTERN-spawn-observability`, output file) is unpinned because `startAction` returns an error under `go test`, and `TestSettle` only calls a fake `RunAction`. A wrong argument vector would ship green.

### [LOW] A failed record write after a successful release reports "not filed"

**Location:** internal/adoptcli/fixloop.go:failedEnd
**Issue:** `fronttask.File` returns an error when only the record write failed, after the entry was created and released and the try count raised. `failedEnd` then tells the operator the fix task was not filed, but the dobby starts it, and a re-run files a second attempt on top of it.

### [LOW] Condition-stop reason can be empty

**Location:** internal/loomshed/darnwrite.go:149
**Issue:** D9 drops `darnStopWayForward` for a condition stop, so the reason is the report alone. A writer that creates both markers and writes an empty report halts the run Stuck with an empty reason.

### [LOW] Green end reads the status of other subjects' children

**Location:** internal/adoptcli/green.go:resumeHaltedChildren
**Issue:** The subject filter runs after `ChildStatus`, so a read failure of another subject's pair is reported in this subject's notice, and each green end reads every pair of the hub.

### [LOW] HubSettleDeps owner read is untested

**Location:** internal/fronttask/hub.go:74
**Issue:** The "on the subject's owner only" rule (D8) is pinned only through a fake; the real owner read through `boardengine.ReadOwners` and `Effective` has no test.

### [LOW] Overview omits the front-task piece

**Location:** docs/overview.md:458
**Issue:** The record asks that the docs follow; the package docs do, but the overview names `internal/fronttask` nowhere.

## Rejected

- Code-tests F3 (dobby wiring in `beamcli/dobbydeps.go` untested): the lens could not confirm the absence, and the `SettleFronts` seam and `Held` read are pinned in `dobbyengine` and `beamcli` tests.
- Code-tests F4 (replay of `front` and `on_land` untested): unconfirmed by the lens, and no replay defect was shown.
- Code-tests F5 (`hubFixDeps` wiring untested): the config read and the vocabulary row are pinned at their sources by `config_test.go` and `TestFailedEnd`; a wiring test would only repeat them.
- Code-affected-set F2 (upsert help text shortened): the cut was forced by the Long cap in a separate commit, and the lens named no wrong statement.
