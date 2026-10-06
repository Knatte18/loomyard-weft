---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    location: _lyx/discussion/decision-record.md Scope and Decision 8
    summary: The loomcli stub-driver tests answer skill loads with the pane unknown-skill notice that Decision 8 removes, and the scope leaves them out
---

### [BLOCKING] loomcli's stub-driver tests depend on the removed pane classifier

**Location:** Scope ("In" lists only `shuttleengine`, `claudeengine`, `orchengine` and `orchcli`) and Decision 8.

**Issue:** `internal/loomcli/integration_driverbootstrap_test.go` and `internal/loomcli/smoke_driverstrand_test.go` launch a shell stub in place of `claude`.
Each stub answers a typed `/<skill>` line with `Unknown skill: <name>`, which `SkillUnknown` reads from the pane to skip the skill at once.
Decision 8 removes that classifier and Decision 1 types one plain message instead of `/<skill>` lines, so neither stub matches any longer.
Each launch would then wait out the load timeout (Decision 7, two minutes by default), past the 30 s bound these tests give `lyx loom start`.
This is the failure that blocked `test-suite-measure` at Publish on 2026-10-06: verify runs the `tmux` tier.

**Fix:** Add both files to Scope, and add an acceptance criterion that their stubs end the load turn under the new protocol, for example by writing a turn-end event, or that their specs set a short `SkillLoadTimeout`.
Both files sit in `internal/loomcli`, whose tests the parallel `test-suite-prune-core` prunes.
Limit the edit to each stub's skill-load answer, so the two runs touch disjoint hunks.
