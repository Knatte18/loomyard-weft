---
verdict: APPROVED
findings:
  - id: F1
    severity: NIT
    class: design
    location: internal/shuttleengine/activity.go:109
    summary: waiterlessRuns keeps the key of a run directory deleted mid-episode, so its recovery line is never logged
    fix: No change needed; the unreadable-record episode set has the same bounded property. Leave the code as is.
    origin: lens:code-correctness
---

The amendment is the parent merge into `internal/shuttleengine/doc.go`.
The diff against the settled patch differs only in that file's context lines; `activity.go` and `activity_test.go` are unchanged since the settle.

### [NIT] A deleted run directory leaves its waiterless episode key behind

**Location:** `internal/shuttleengine/activity.go:109`
**Issue:** A run directory removed while its waiterless episode is open is never listed again, so the key stays in `waiterlessRuns` and no recovery Info is logged.
The leak is bounded by the number of run directories and matches `unreadableFiles`, so it is an observation, not a defect.

## Rejected

- Merge damage or contradiction in `doc.go`: the parent's usage-limit sentences sit in the held-turn-end section near line 104, and the task's paragraph sits in the Agent activity section near line 427, so the two do not touch.
- Missing dedicated test for the separate episode key space: the three-read phase of the new subtest would log three Warns if the key were shared, so it already pins it.
- Snapshot `log` in the new subtest taken before the later phases: harmless, since `run-b` and `run-c` never change.
