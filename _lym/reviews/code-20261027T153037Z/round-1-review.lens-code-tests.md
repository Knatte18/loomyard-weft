```yaml
lens: code-tests
outcome: reported
```

Task: lens code-tests, listing the test that pins each behaviour the diff changes. No tool calls; the tests were not re-run.

**Per behaviour: the test that pins it**
- Darn row order and edges: `wantDarnProducerTable` and `TestNewDarn_Routing` in `internal/loomrecipe/darnrecipe_test.go`.
- `entry_gate: drift.enabled` on darn's Drift-Await: the darn row of `TestDriftRows_EntryGate`.
- Row indexes shifted in darn: the darn case of `TestEntryNumbers`, and `TestRenderChecklist` in `internal/loomcli/checklist_test.go`.
- DriftBaseline refuses a `capture` key naming the way forward: the `BaselineCaptureKey` row in `TestDriftEntries`; the substring `entry_gate` is adequate.
- Baseline with the capture lock held returns Done without waiting: "a held lock is left running" in `TestDriftBaseline`.
- Baseline starts a detached capture: "partial starts the detached capture".
- Windows breakaway fallback: "a breakaway refusal falls back to the foreground capture" and "a failed foreground fallback is Stuck naming capture and defer".
- `CountedCapture` counts only with `awaited_at`: `TestCountedCapture`.
- Drift-Verify judges by `awaited_at` on every recipe: the "no counted capture is a decision halt naming the waiver under every scope" subtest.
- `lym drift waive` refusing a counted capture: existing `Waive` rows in `verbs_test.go` (by inference; not in the diff).
- `driftcli/wiring.go` seed read dropped: no test, none needed.
- `reportengine` darn phase for Drift-Await: `darnRowPhases` in `timing_test.go`.
- Removed modes: rows gone; `onSleep` and `sleeps` in the baseline fixture are still used by `await_test.go`.

**Test economy**
- `BaselineBoardEntry` differs from `BaselineDecisionRecord` only in subject and keeps the coverage the removed wait row carried; not a duplicate.
- `BaselineUnknownKey` merged into `BaselineCaptureKey`; unknown-key refusal still covered.

No findings.

Tool calls: none.
