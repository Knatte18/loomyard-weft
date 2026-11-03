```yaml
lens: fix-affected-set
outcome: reported
```

Task restated: I reran the set searches for F1–F9 against the fixed tree and checked that each fix covers every member of its set.

No findings.

Per finding:
- **F1:** `BaselineInputs{` is built in two places, `internal/loomcli/driftwiring.go:126` (now sets `AnchorDir`) and `seats_test.go:58` (the helper now sets it). `captureReading` has one caller, which passes the new arguments.
- **F2:** The remaining `AnchorPath()` uses in `battencli` tests concern the pair's local layer and run-record paths (`lifecycle_integration_test.go:1847` and `:1901`, `dobbynotice_test.go:73`). They are not `PairDir` assertions, so no stale `PairDir` check is left.
- **F3 and F8:** `StatCommandDir` has three call sites, the runner, the drift `CheckDirs` and the gate `CheckDirs`, and each is covered by a test row. `configreg` wires `CheckDirs` for drift and gate only. `checkHubModule` is the single editor and `--set` path.
- **F4:** There are five request constructions in non-test code. The two engine `stepRun` sites are pinned by the new `RunCompare` and `RunSideCapture` rows, and the existing `RunCapture` row already covers the third. The four request helpers (two in `driftcli`, two in `loomcli`) are pinned by the new wiring tests. No inline `CaptureRequest{` or `CompareRequest{` construction remains outside the helpers.
- **F6:** All three `Site:` values use the `site*` constants, and `missingDirRefusal` switches on `run.Site`. No leftover `run.Name == stepCapture` branch. The rows pin the side-capture and compare refusals, and the existing capture row pins the capture one.
- **F7 and F9:** The base statements in `driftcli/doc.go:14` and the request doc comments agree with the worktree-root base. A grep for the old anchored-directory base wording found nothing stale in `internal` or `contracts`. The `ResolveCommandDir` comment and `gateslot/doc.go` no longer claim one rule.

Findings left unchecked: none.

Tool calls made:
1. Bash grep of F1, F4, F6 and F7/F9 sites, because my context did not hold the repo-wide caller and construction sets.
2. Bash grep of F2, F3 and F8 sites, for the same reason.
