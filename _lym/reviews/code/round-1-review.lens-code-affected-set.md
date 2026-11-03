```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens, covering what the diff forces outside the lines it touches.

**Per change, the set found, covered or not**

- **New row `Drift-Recheck` (`NameDriftRecheck`, engine `DriftRecheck`, registry entry).**
  - Covered: both recipe yamls, `loomshed.go`, `interruptpolicy.go`, `reportengine/timingspans.go` (loom and darn phase maps), and `registry.go`.
  - Covered: tests, namely the `loomrecipe` shape, routing, progress, sequence, entry-number and drift entry-gate tests, `tapestry-rows-before-motif.yaml`, `loomcli/checklist_test.go`, `reportengine/timing_test.go`, `shedrecipe` entries/registry/conformance tests, and `battencli/wire_seams_test.go`.
  - Covered: docs, namely `docs/overview.md`, `loomrecipe/doc.go`, `shedrecipe/doc.go`, and `driftengine/doc.go`.
  - **Not covered:** `internal/reportengine/doc.go:130,132` (F1).
  - **Not covered:** `internal/driftengine/template.yaml:1` (F2).
  - **Not covered:** the tapestry recipe comment (F3).
- **`Code-Bouncer` `on_done` changed from Describe to Drift-Recheck.**
  - Covered: both recipes, the fixture yaml, and the routing and progress tests.
  - Covered: `loomshed` and `gateslot` docs and the PATTERN files name no Bouncer-to-Describe edge.
  - **Not covered:** the tapestry comment about a run inside the perch exiting to Describe (F3).
- **New `Deps` seams `CaptureAccepted` and `CompareAccepted`.**
  - Covered: `loomcli/driftwiring.go`, `envkit`, the `shedrecipe` seam checks, `battencli` intentionally-nil table, and the verify and recheck fixtures.
  - No other `Deps` constructor found in the grep.
- **New `ScopeRecord.AcceptedProfileHead` key, cleared in `Accept` and `Withdraw`.**
  - Covered: `records.go`, `verbs.go`, `verbs_test.go`, and `driftengine/doc.go`.
  - Not read, so not checked: `verdict.go` and `records_test.go`. They read the scope record, but the diff changes neither.
- **New `Paths.AcceptedDir` and `siteAcceptedCapture`.** Covered: `driftengine/doc.go` and `paths.go`. The ephemeral `.lym/drift/accepted` path stays inside `lymdirs`' declarer.
- **`RunCompare` `ProfileDir` field and `RunAcceptedCapture`.** Covered: `driftengine/doc.go`, `capture_test.go`, and `compare_test.go`.
- **PATTERN.md and `pattern/`.** The diff touches neither, and the directive declares no PATTERN changes. No entry is violated. `PATTERN-gate-slot.md` and `PATTERN-test-speed.md` list Drift-Verify only as a gate site, and Drift-Recheck takes no gate.
- **Commit messages.** Neither carries a `Pattern-Departure:` trailer, and none is needed.

**Findings**

- **F1 — LOW, consistency** — `internal/reportengine/doc.go:130` and `:132`.
  - The doc lists each phase's rows as the code leaves them. The tapestry `webster` list and the darn `review` list stop at Drift-Verify and omit Drift-Recheck.
  - `timingspans.go:102,120` now maps it (to `PhaseWebster` and `PhaseReview`), so the doc contradicts the code.

- **F2 — LOW, consistency** — `internal/driftengine/template.yaml:1`.
  - The `enabled` comment still says the switch turns on "Drift-Baseline, Drift-Await and Drift-Verify".
  - Drift-Recheck now carries `entry_gate: drift.enabled` too, so the user-facing config template is stale.

- **F3 — LOW, consistency** — `contracts/recipes/tapestry-recipe.yaml:74-76`.
  - The header comment says a run in flight "inside the perch exits to Describe without Drift-Verify".
  - The perch now exits to Drift-Recheck. For such a run Drift-Recheck sends it back to Drift-Verify when there is no compare recorded, or compares only if one was recorded.
  - The comment's `lym loom goto --to Drift-Verify` advice may still hold, but its stated destination does not.

**Tool calls**
1. `grep` for `NameDriftVerify` and `NameDriftAwait` consumers, plus every file mentioning `Drift-Verify`. My context lacked the full set of consumers and docs outside the diff.
2. `grep` for Drift-row mentions in `loomshed`, `driftcli`, `loomcli`, `reportengine`, `shedadapters`, `driftengine` and `gateslot` docs and code. I needed to see which describe the row set.
3. `sed` of `reportengine/doc.go` lines 130-132, `grep` for every `Drift-Recheck` mention outside the diff's own packages, and `grep` for Drift in `PATTERN.md` and two pattern files. These confirmed the stale doc and that no PATTERN entry is touched.
