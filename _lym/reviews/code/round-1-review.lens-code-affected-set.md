```yaml
lens: code-affected-set
outcome: reported
```

**Task:** code-affected-set lens on the drift-scope-in-run-settings diff: for each change, what it forces outside the touched lines, and whether the diff covers it.

**Per change: set found, covered or not**
- `drift.yaml` keys `scope` and `skip`:
  - **Covered:** the template, the `ConfigSpec` `CategoryNone` registration, `doc.go`, and the `wantLocalKeys` list in `configreg_test.go:80`.
  - **Covered:** `PrepareRunSettings` and `ValidateLocal` accept a `{"drift": {...}}` object because the keys are registered.
  - **Not covered:** the Drift-Verify stencil (finding 1).
- `LoadConfig` refusals:
  - **Covered:** the new `TestLoadConfig` rows.
  - **Covered:** the `doc.go` and `LoadConfig` doc-comment lines.
- `DriftBaseline.drive` with a configured scope:
  - **Covered:** the `Call` doc comment, the `doc.go` paragraphs on the producer and on the scope record's writers, `driftcli` `Long` and the `scope` help, the overview paragraph, and the orch-role contract.
  - **Not covered:** the Drift-Verify stencil, which reads the generated `baseline.md` (finding 1).
- `baseline.md` file shape (new generated report):
  - **Not covered:** `contracts/stencils/drift/drift-template-verify.md:23` still describes the file as "The baseline agent's report with the drift it expected: `{{.baseline_path}}`".
  - **Searched:** other `.md` files for "baseline agent" and for `verify_max_runs`. They turned up no other reader of the report or of the key list.
- Constraint entries:
  - **Nothing to land:** I found no `PATTERN-` entry that governs this change, and the commit declares no PATTERN change.
  - **No departure to excuse:** the commit has no `Pattern-Departure:` trailer.
  - **Checked, no finding:** `PATTERN-config-layers` is met, since the change adds keys to the module spec and `configengine` does the loading. `PATTERN-sole-parsers` and `PATTERN-stencil-ownership` are untouched.

**Findings**
1. **LOW, class `consistency`:** `contracts/stencils/drift/drift-template-verify.md:23` calls the baseline report "the baseline agent's report with the drift it expected". Under a configured scope the diff generates that file with no per-test expectation (`baseline.go:168-176`).
   - Drift-Verify's agent gets no signal that no agent wrote the file or that no drift was predicted.
   - The line is neutral enough to leave as is. Because the diff makes this file's content differ, a one-clause stencil note is the fix. The diff did not touch the stencil.

**Tool calls**
1. `grep -rln "verify_max_runs"` over `*.md`, `*.yaml` and `*.go`, and `grep -rn "baseline agent\|Drift-Baseline agent"` over `*.md`. My context had no search of the other docs, stencils or fixtures for the drift key list and the baseline report.
