---
round: 1
focus:
  - "cmd/lyx/testdata/test-budget.yaml: the parent raised several package rows and the task added a `tools/forkaudit` row; settle whether the merged file still matches the test-budget rule and the repo's budget scan, and whether the task's `tools/forkaudit` row is consistent with how the parent changed the file's rows."
  - "tools/forkaudit against the parent's new code: settle whether the task's tool still follows PATTERN.md and any `pattern/` invariant the parent's commits added or changed under `tools/` or in the packages it reads."
  - "Symbols or contracts the parent changed (for example in `internal/shuttleengine`, `internal/websterengine`, `internal/webstercli`, `internal/statuspane`) that the task's tool or its tests read, call or assume; settle whether any merged call site now does something neither side meant."
---

The sides overlap only in `cmd/lyx/testdata/test-budget.yaml`, in disjoint hunks, and no package overlaps.
The overlap is therefore the primary lead, and the other two entries check for silent semantic conflicts across the non-overlapping files.
