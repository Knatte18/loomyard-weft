---
round: 1
focus:
  - "Tests the parent added or changed under `internal/websterengine`, `internal/burlerengine`, `internal/orchengine`, `internal/treadleengine` and `internal/loomcli` (`git diff 3394b072d^1 3394b072d`): does any pin a stencil's static prose, which the task's new `PATTERN-test-economy` bullet forbids, or duplicate a marker-set or Fill round-trip test the task kept in `contracts/stencils/markers_test.go`?"
  - "Stencils and rubrics under `contracts/stencils/loom/` that the parent edited while the task trimmed their tests (`rubric_test.go`, `markers_test.go`, `internal/websterengine/template_test.go`, `internal/treadleengine/template_test.go`): do the task's remaining marker and position pins still match the parent's new markers and Go-filled fragments?"
  - "Symbols the task's trimmed tests or helpers reference that the parent renamed or moved in the overlapping packages (`cmd/lyx`, `internal/loomcli`, `internal/orchengine`): did anything merge cleanly yet call a changed signature or contract?"
  - "The changed `pattern/` files (`PATTERN-creel-store.md`, `PATTERN-shuttle-provider-seam.md`, `PATTERN-daemon-wakeups.md`, `PATTERN-pane-text-injection.md` and the rest listed): does any task-edited test or helper break an invariant the parent added there?"
  - "Test economy across the merge: a behavior the parent's new code introduced in the overlapping packages that the task's deletions left with no covering test, and a parent test that now duplicates a surviving task test (name the covering test)."
---

Seed focus drawn from the diffstat of both sides: the task is test-only plus rubric, stencil and `PATTERN-test-economy` text, and the parent is a large feature merge touching the same packages' tests.
