---
verdict: APPROVED
---

The amendment round settles the three files the parent and the task both changed: `internal/driftcli/wiring.go`, `internal/loomcli/driftwiring.go` and `internal/loomcli/wiring.go`.
The parent's change to them is the `ReadJSONNoMkdir` to `ReadJSON` swap.
The task's removals of the `awaitRow` parameter, `Deps.AwaitRow`, `VerbStatus.AwaitRow` and the seed read in `verbStatus` survive the merge intact.

- `verbStatus` reads status through `state.ReadJSON`, carries no seed or `AwaitRow` read, and its doc comment matches its body, so `lym drift` verbs behave the same for loom and darn runs.
- `driftDeps` and its call site in `wire` carry no `awaitRow` argument, and the doc comment names no such parameter.
- A repo-wide search outside `_lym` finds no `AwaitRow`, `WaitCapture`, `awaitRow` or `ReadJSONNoMkdir`, and no recipe, doc or test uses `capture: wait`.
- `go build` and `go vet` of `driftcli`, `loomcli`, `driftengine` and `shedrecipe` are clean.

## Rejected

- The comment above `state.ReadJSON` in `segmentBounces` (`internal/loomcli/wiring.go`) about a run directory not yet created sits beside the parent's swap: it is the parent's text, not a line the task changed, so it is outside this amendment.
- The dead `onSleep` scaffolding in `internal/driftengine/baseline_test.go` (a lens-code-tests LOW) belongs to the settled remainder, which this round reports nothing about.
- The `BaselineUnknownKey` row replaced by `BaselineCaptureKey` in `internal/shedrecipe/entries_drift_test.go` (a lens-code-tests NIT) is the same: settled remainder, and both rows reach the same branch.
