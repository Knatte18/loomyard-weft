---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: internal/beamcli/dobbydeps_integration_test.go:25
    summary: no test pins that hubDobbyDeps fills the TaskLimit seam, a D6 acceptance criterion
    fix: In the existing integration test of hubDobbyDeps in internal/beamcli/dobbydeps_integration_test.go, assert the returned dobbyengine.Deps has a non-nil TaskLimit; never call the seam, since it reads the host's real cgroup.
  - id: F2
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: internal/dobbyengine/starts_test.go:308
    summary: nothing pins that the limit is read right before each start rather than once per tick
    fix: In TestStartReady_TaskLimitHold add a subtest whose TaskLimit fake returns a free reading on its first call and a near reading on later calls, with two ready tasks and max_runs above 1, and assert only the first task spawns and one hold line is logged.
---

The diff makes the change the amended board entry asks for.
The reader walks the dobby's own cgroup ancestry and picks the level with the fewest free tasks.
It skips `max`, missing files and malformed numbers per level, and it stops at the root.
The hold sits right before a task's first pass and writes no start record, notice or backoff.
`task_headroom` defaults to 500, accepts 0 and refuses a negative value naming the key.
Passes of runs already started, the dobby view and hand `lym batten run` are untouched, and the docs land in the same diff.
No `Pattern-Departure:` trailer is needed and no PATTERN entry is declared changed.
The change description's claims match the diff.

### [LOW] No test pins the hubDobbyDeps wiring

**Location:** internal/beamcli/dobbydeps_integration_test.go:25, internal/beamcli/dobbydeps.go:146
**Issue:** The decision record's D6 acceptance criterion says `hubDobbyDeps` fills the seam.
Dropping `TaskLimit: proc.ReadTaskLimit` breaks no test, and the hold would then never fire in production.

### [LOW] No test pins the per-start reading

**Location:** internal/dobbyengine/starts_test.go:308, internal/dobbyengine/starts.go:99
**Issue:** The fake reading is constant, so a version that read the limit once per tick would pass every row.
The decision record's D4 asks for the check right before each task's first pass.

## Rejected

- The code-tests fork's finding that no row holds, resumes and holds again: `taskLimitHolds` assigns `d.taskLimitHeld = held` on every call, so the flag cannot stay set after a resume, and the first subtest already pins the resume line once.
- The code-tests note on a missing `task_headroom: 0` config row: the fork itself withdrew it, since the shared negative check is covered by the `max_runs: 0` row.
