---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    location: _lym/discussion/decision-record.md, D5 Bound
    summary: D5's bound says a live older lym process only refuses a second overview add, but the pawl run's old-build driver runs the old ensureRunsStrand in-process, whose ReplaceStrand kills the runs strand before the refused add, or replaces the status strand again
    fix: In D5's Bound, replace the sentence on an older lym process with one naming the old-build pawl driver, which calls orchcli.BeamUp in-process at its Beam-Up row; state that while it lives it can drop the runs strand (old ReplaceStrand removes it before refusing the overview add) or replace the status strand with the runs strand, that a hand lym loom start keeps that driver, and that the runs strand comes back at the next lym beam start under the new build (D4).
    origin: lens:bound-breaker
  - id: F2
    severity: LOW
    class: scope
    location: _lym/discussion/decision-record.md, Acceptance criteria, D2 bullet
    summary: The D2 criterion covers the relevant window and follow target but not the viewed-window-ending path, where a viewer on one of two overview windows that ends must land on the other overview window
    fix: Append to the D2 acceptance bullet, or add a D2 bullet - with two overview windows and no live agent, a viewer on the lower-index overview window that ends moves to the other overview window, never the shell.
    origin: handler
  - id: F3
    severity: LOW
    class: consistency
    location: _lym/discussion/support-log.md, Question ledger, Q5
    summary: Q5 says an open risk was recorded for task sessions, but the record's Open risks reads None and Turn 4 (leanness 6) says that line was dropped for the auto-mode assumption
    fix: In support-log.md Q5, replace "open risk recorded for task sessions (D3)" with "the task-session rename recorded as an auto-mode assumption (D3)".
    origin: handler
  - id: F4
    severity: NIT
    class: consistency
    location: _lym/discussion/decision-record.md, D5 Bound
    summary: D5's bound states the old-key status strand ties the runs strand by opening order, but not that D2's fallback then lands on the pawl checklist instead of the beam overview, against D2's rationale
    fix: In D5's Bound, after the tie sentence, add - while the status strand opened first, the focus fallback of D2 lands on its window rather than the beam overview, until that start replaces it.
    origin: lens:interactions
---

### [LOW] D5's bound understates an old-build pawl driver

**Location:** `_lym/discussion/decision-record.md`, D5 Bound.
**Issue:** The pawl run's Go driver calls `orchcli.BeamUp` in-process (`internal/loomcli/pawlwiring.go:106-107`), which runs `ensureRunsStrand` (`internal/orchcli/start.go:483`).
A driver started by the pre-change build and still live after the upgrade runs the old `ensureRunsStrand` (`internal/orchcli/runs.go:84-91`).
Facing a runs strand whose sidecar names the new build, it calls the old `ReplaceStrand`, which removes and kills the runs strand (`internal/reedengine/replace.go:64-71`) before `addStrandLocked` refuses the overview add (`internal/reedengine/strand.go:404-409`), so the beam overview is lost.
Facing no runs strand, it replaces the status strand again.
The record says only that such a process "refuses a second overview add of its own", and D5's healing start keeps a live driver.
Severity is LOW, not the lens's MEDIUM: the state lasts only through an upgrade and the next `lym beam start` restores it, so the fix is a corrected bound, not a design change.

### [LOW] D2 acceptance misses the viewed-window-ending path

**Location:** `_lym/discussion/decision-record.md`, Acceptance criteria.
**Issue:** D2 names a viewed window ending as one of the fallback's routes, and `RemoveStrand` and reconcile reach it through `selectRelevantWindowIfCurrentLocked` with an ending set (`internal/reedengine/ownwindow.go:514-522`).
With two overview windows, the ending filter in `agentWindowPanes` must still yield the other overview window.
No criterion would fail if it yielded nothing or the ending window, so the focus directive's demand that D2's ending path have a failing criterion is unmet.

### [LOW] Question ledger Q5 contradicts Open risks

**Location:** `_lym/discussion/support-log.md`, Q5.
**Issue:** Q5 ends "open risk recorded for task sessions (D3)", while the record's `## Open risks` reads `None.` and Turn 4's leanness 6 says the line was dropped and the auto-pick kept.
The record is right; the ledger is stale.

### [NIT] D5's tie bound omits its focus consequence

**Location:** `_lym/discussion/decision-record.md`, D5 Bound.
**Issue:** In a session where an older lym put the status strand in the overview with no key and the new `lym beam start` then adds the runs strand at key 0 (D4), the two tie and the status window sits first.
D2's lowest-index fallback then picks the checklist, against D2's rationale that the lowest-index overview is the beam overview.
The bound states the order, from which this follows, so this is a NIT.

## Rejected

- Focus item 1, D1 bound for a spawner with no or another explicit name: not a defect. `lym reed add` (`internal/reedcli/add.go:77-82`) and `lym shuttle run` (`internal/shuttlecli/run.go:113`) both refuse the overview anchor, and only `internal/loomcli/bootstrap.go` and `internal/orchcli/runs.go` set it, each under an explicit name.
- Focus item 2, D3 and the old name or `KeepViewer`: not a defect. No code looks a window up by the name `overview` (`overviewWindowName` is only the default title), and what `KeepViewer` guarded, the checklist never taking the viewer, carries over through the Constraint that an overview window is never selected when it opens (`internal/reedengine/spawn.go:280-286`).
- Focus item 3, the plan-writer note leaving the order key's source open: not a defect. It is an implementation choice, not a fact the design rests on: `openOwnWindowLocked` already holds reed state and the pane-to-window map (`internal/reedengine/spawn.go:261-283`), so either source works, and both read an older lym's window as key zero.
- Focus item 4, D5's sidecar claim: not a defect. `StrandCurrent` keeps a strand only when the sidecar's build equals the running build (`internal/statuspane/build.go:80-82`), so the first start under the new build replaces it, and a session without a status strand takes the add path; D5 names this mechanism.
- Focus item 6, relocation: no deliberation narrative sits in the record, and the support log's Turn 4 fixes and Rejected alternatives each carry a constraint the record already states (D1's tie rule replaces the rejected refusal on equal keys).
- The existing-paths and walkthrough lenses returned no findings; their traces agree with the handler's reading.
