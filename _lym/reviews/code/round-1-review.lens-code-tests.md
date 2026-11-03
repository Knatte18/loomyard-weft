```yaml
lens: code-tests
outcome: reported
```

Lens: code-tests. For each behaviour the diff adds or changes, this lists the test that pins it, or none.

**Behaviour → pinning test**

1. **The capture child starts in the anchored directory (#23).**
   - Pinned at the engine seam by `TestStartCapture` in `spawn_test.go` (about line 96), which asserts `spawns[0].dir == AnchorDir`.
   - The engine tests use the same directory for the anchor and the worktree root (`AnchorDir: root`, `capture_test.go`). The join onto `AnchorDir` is therefore never told apart from the join onto the root.
   - The wiring that feeds the engine is not pinned. That is `w.anchor()` in `driftcli/cli.go` and `location.AnchorPath()` in `loomcli/driftwiring.go`, for the child start, `RunCapture` and `RunCompare`. Every drift-wiring fixture I found uses `Anchor: "."`, so reverting to `WorktreePath()` fails no test. **None** (F1).
2. **Capture and compare commands run in the anchored directory.**
   - `TestRunCompare` asserts `seen.txt` under `AnchorDir`, which pins the join in `runStep`.
   - Subdirectory-anchor wiring is not covered. This is the same gap as F1.
3. **`gateslot.Holder.Worktree` now carries the anchor dir instead of the worktree root (`steprun.go:149`).** No test asserts it. The lock-held tests only pass the value in. **None** (F2).
4. **`passWhenSwitchedOff` in Drift-Baseline.**
   - Pinned by the row "drift switched off passes a run halted on a failed capture, with no agent, capture or commit" in `TestDriftBaseline`.
   - A config load failure is pinned by the row "a drift config that fails to load is Stuck naming lym loom resume".
5. **`passWhenSwitchedOff` in Drift-Await.** Pinned by the row "drift switched off is Done with no commit, whatever the capture" in `TestDriftAwait`. The new nil-`Config` seam check is pinned by the `AwaitNilConfig` row in `entries_drift_test.go`.
6. **`passWhenSwitchedOff` in Drift-Verify.** Pinned by the subtest "drift switched off passes a run with no counted capture…". The existing rows were updated to `Enabled: true`.
7. **The flag is re-read at every pass.** Each pass-through test uses a fixture whose row the run is already in (scope seeded, capture failed). No test flips the flag between two passes of one producer. The per-pass read follows from the call order, so I count it as pinned by those rows.
8. **A capture start error is now a capture halt.** Pinned by the baseline row "another start error is a capture halt…". It asserts `Stuck`, `CaptureHaltPrefix`, `child.log`, `lym loom resume` and `lym drift defer`.
9. **`Defer` is admitted at that halt.** Pinned by `wantDeferrable`, which calls `Defer` at a blocked Drift-Baseline with the pointer reason.
   - The check runs only on the start-error row. The other capture-halt rows (failed capture, refused precondition) are not checked for deferral, but `Defer` itself is unchanged in the diff.
10. **`startFailure` wording without and with `FallbackDetach`.** Pinned by the `failures` table in `TestStartCapture`.
11. **`RecordScope` admission.** The agent records at a running Drift-Baseline and the parent at a blocked Drift-Baseline or Drift-Await.
    - Covered by the rows "the parent replaces full with off…", "the parent records a scope at a Drift-Await halt", "wrong row" and "a halt at another row".
    - The agent's second call stays refused, pinned by "already recorded".
12. **`RecordScope` refusals.** Pinned by the rows "a replacement while a capture runs" and "a replacement after the baseline wait settled".
13. **`RecordScope` drops the capture when the new scope compares a test the old one skipped.** Pinned by three rows: drops (`partialA` → full), keeps (more skips) and keeps (to off).
14. **A replacement keeps the record's other fields** (`cur.Scope, cur.Skip, cur.RecordedAt = …` on a copy of `cur`). The rows assert only scope, skip and `RecordedAt`. Nothing checks that fields such as `Deferred` or `WaivedReason` survive. **None** (F3).
15. **The `capture_command` marker and `captureReading`.**
    - The declared-command rendering is pinned by `TestSeatTables`, which looks for the command and its `run in .` directory in the prompt.
    - The undeclared rendering is pinned by `TestSeatTables`, which looks for "The capture command is not declared yet".
    - The marker list is pinned by `markers_test.go`.
    - The stencil prose is exempt.
16. **The `scopeCmd` help text and the `Drift.Config` seam in `entries_drift.go`.** The seam is pinned by item 5. The help text is static.

**Findings**

- **F1 — MEDIUM, design.** `internal/loomcli/driftwiring.go` (the `StartCapture`, `RunCapture` and `RunCompare` requests) and `internal/driftcli/cli.go` (`w.anchor()` at the compare, side-capture, `startMainCapture` and capture-child sites). Nothing pins that the production wiring hands the engine `AnchorPath()` rather than the worktree root. This is the #23 fix itself. The engine tests use a root equal to the anchor, and the wiring fixtures in `driftcli/wiring_test.go` and `capture_integration_test.go` use `Anchor: "."`. Evidence: `capture_test.go:41-42`, `driftcli/wiring_test.go:34`, `driftcli/capture_integration_test.go:115`. A row with a subdirectory anchor, asserting the child's or command's cwd, would fail on a revert.
- **F2 — LOW, design.** `internal/driftengine/steprun.go:149`. `gateslot.Holder.Worktree` changed from the worktree root to the anchor dir, and no test asserts what holder is recorded. Either pin it, or restore `Worktree` to the worktree root if that field is meant to name the worktree. The doc comment on `Holder.Worktree` says "the worktree".
- **F3 — LOW, design.** `internal/driftengine/verbs_test.go`, the `TestRecordScope` replacement rows. A replacement keeps the record's other fields (`verbs.go:466`), but no assertion checks that fields such as `Deferred` or `WaivedReason` survive. Add one assertion to an existing replacement row.

**Tool calls**
1. `grep -rln` for anchor and scope symbols in the `driftcli` and `loomcli` tests: to find tests that pin the wiring, which my context lacked.
2. `grep` for `anchor` in `driftcli/capture_integration_test.go` and `wiring_test.go`, and for `driftDeps` in the `loomcli` tests: to check whether any fixture uses a subdirectory anchor. All use `Anchor: "."`, and `loomcli` has no `driftDeps` test.
