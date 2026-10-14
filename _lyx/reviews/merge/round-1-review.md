---
verdict: APPROVED
findings:
  - id: F1
    severity: NIT
    class: consistency
    location: internal/yamlengine/set.go:53
    summary: the merged SetValues and SetSparse doc comments state the null-tag rule for every scalar, while a key at a scalar-or-list open-map path always becomes a string, so a spelled null there is a string
    fix: in the SetValues doc comment in internal/yamlengine/set.go (the line starting "A scalar keeps its leaf's tag") and the matching line in the SetSparse doc comment in internal/yamlengine/layers.go, scope the sentence to a scalar leaf outside a scalar-or-list open-map path, for example by starting it "A scalar leaf outside those paths keeps its tag", so the two sides' rules no longer read as contradicting for a pair like discussion_review=~.
---

### [NIT] Null-tag rule and scalar-or-list rule are stated without saying which wins

**Location:** `internal/yamlengine/set.go:53`, `internal/yamlengine/layers.go:151`
**Issue:** The task added `scalarSetTag`, under which a spelled null gives a scalar the null tag.
The parent added `scalarOrListNode`, which runs first in both `SetValues` and `SetSparse` for a key at a declared scalar-or-list open-map path and writes any non-bracketed value, `~` included, as a string.
The two paths never meet today: loom's null budget keys are not in `ConfigOpenMaps`, and the open-map keys have non-null template values.
The merged doc comments still state both rules without saying which applies to such a key.

## Checked and found consistent

- `SetValues` and `SetSparse`: a list key, a null-template leaf and a spelled null each reach one branch, and the parent's branch is taken first. The task's tag change touches only the plain-leaf fallthrough, so the two edits do not interact.
- `wire.go` and `innerrun.go`: the task's `ArbiterStop` seam and its nil default in `NewInnerRun` sit beside the parent's `OpenTerminalOnce` and `spawnPairPush` changes without sharing any state. The task's seed path does not touch `PushSeed`, so the parent's detached-push contract in `deps.go` holds.
- `PATTERN-fabric-git`: no call site the task added or changed calls `PushAnchored`, `Add` or `SpawnPairPush`.
- `PATTERN-no-denied-recovery` and `PATTERN-daemon-wakeups`: the task added no `systemctl` form to a stencil, spec or Go string literal, and no unbounded wait loop. The hold runs inside the existing Run-Shed wait.
- `claims_test.go`, the orch stencil and `refusal-spec.md`: the parent's hibernate claim and paragraph and the task's driver and arbiter claims and rows do not touch each other. The driver stencil contains no hibernate wording.
- `test-budget.yaml`: the merged rows are at or above the top-level test counts of every package both sides touched, and `TestTestBudget_PackagesStayWithinBudget` passes on the merged tree.
- `docs/overview.md`: the merged module table and sections carry the parent's hibernate modules and the task's text, with no lost hunk.
- Test economy: the task's new `yamlengine` rows and `lifecycle_integration_test.go` assertions cover null and park-marker behavior the parent's tests do not, and the reverse. The hold's creation-time interaction with the terminal open and the pair push is only code-adjacent, since the two live in different rows, so no uncovered shared behavior exists.
