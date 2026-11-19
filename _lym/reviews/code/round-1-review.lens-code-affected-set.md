```yaml
lens: code-affected-set
outcome: reported
```

code-affected-set lens: what the diff's changes force outside the lines it touches.

**The set, per change**

- **`shedrun.WriteLandedCommit` / `ReadLandedCommit` signature and file shape** (a second line `empty-diff`, and a `LandedCommit` struct in place of a string):
  - Callers: `battencli/dobbyreads.go:231-236` (covered), `loomcli/landingdeps.go:241` (covered).
  - Tests: `shedrun/landed_test.go`, `beamcli/dobbypairanchor_test.go:283,312`, `loomcli/landingdeps_integration_test.go:334` (all updated).
  - Docs: `shedrun/doc.go:47-49` is updated. `beamcli/doc.go:483` says DobbyRuns "reads the landing commit through `shedrun.ReadLandedCommit`". It is not wrong, since the read now also carries the flag. Not a finding.
  - An older binary reading a two-line file: its `TrimSpace` keeps `<sha>\nempty-diff` as the commit, and `landNoticeLine` cuts that to 12 characters, so a full SHA still prints right. Covered.
  - The file is plain text, not JSON, so PATTERN-tolerant-records does not apply.
- **`Deps.RecordLanding` signature `(commit string, emptyDiff bool)`**:
  - Production caller: `finalize.go:741`. Wiring: `landingdeps.go:236`.
  - Tests: `finalize_test.go` (every assignment updated) and the loomcli integration test.
  - Docs: `landingshed/deps.go:241-247` and `landingshed/doc.go:564-569` (both updated).
  - Covered.
- **`RunView.EmptyLanding` and the dobby notice line**:
  - Code: `dobbyengine/passes.go`, `landnotice.go`.
  - Docs: `dobbyengine/doc.go:439-442` (updated).
  - Tests: `landnotice_test.go`. `landnotice_creel_test.go:66` builds a RunView without the flag, which is still valid.
  - No stencil or orch text under `contracts/` quotes the `landed as` line; grep found none.
  - The commit message and summary quote `landed with an empty diff: no file of its parent branch changed, which stays at <commit>`, which matches `landnotice.go:318`.
  - Covered.
- **darn.yaml key `report_dir`**:
  - `template.yaml:6`, `ConfigSpec` category, `configreg_test.go:151` local keys, `darnengine/doc.go:12-16`, `config_test.go` (all covered).
  - PATTERN-config-layers' rule that every template key is classified in the same change holds.
  - Not a change of meaning, so no reinterpretation is needed.
- **Runprofile `docs` block gains `report_dir`**:
  - `runprofile/template.yaml:92` and `runprofile_test.go:232` (covered).
  - The summary says reconcile carries the key into a hub's `profiles.yaml`. `runprofile/doc.go:39` bears that out. Covered.
- **New stencil `darn-fragment-report` and marker `report_directive`**:
  - The registry entry in `stencils.go` and the embed are covered.
  - `darn-template-write.md` header comment is updated (marker list and `make_change` line note). It is read through `stencilstore.ReadForEngine` (PATTERN-stencil-ownership holds), and it carries no engine-specific term.
  - `markers_test.go:21-66` is a hand-picked list that omits `darn-template-write` too, so leaving it alone is not a gap.
- **Behaviour change: a docs-profile darn writer now commits a report file under `docs/research/`**:
  - Affected reader: the darn recipe's Code-Review rubric. `contracts/stencils/loom/loom-rubric-code-review.md:65-66` says "a change the diff makes that the answer key does not license" is a finding.
  - Neither that rubric nor anything else tells Code-Review that a `report_dir` report is licensed. **Not covered**; see finding 1.
- **Declared PATTERN changes**: the directive declares none, and the diff touches no `PATTERN.md` or `pattern/` file. No finding.
- **Commit messages**: no `Pattern-Departure:` or `Loosened-Check:` trailers, and none is needed. No test assertion was loosened: the `landed_test` and `landingdeps` asserts got tighter. The message claims match the diff. No finding.

**Findings**

1. **MEDIUM, consistency.** `contracts/stencils/darn/darn-fragment-report.md:13-15` against `contracts/stencils/loom/loom-rubric-code-review.md:65-66`.
   - The report passage tells every run with `report_dir` set ("This run lands what the task finds as a committed report … a run whose task asks for no other change commits the report alone") to commit `docs/research/<slug>.md`. For the `docs` profile that is every docs run, including tasks such as comment trims whose board entry asks for no report.
   - The darn Code-Review rubric the same run is judged by still flags "a change the diff makes that the answer key does not license" as a finding. Nothing in the rubric or the passage licenses the report file.
   - Result: a Code-Review round can flag the config-mandated report as unrequested scope, and the fixer can then delete it.
   - Fix: either word the passage so the report is written only when the task asks for findings or a report, or have the rubric, or the scope a darn run is judged by, admit a report under the configured `report_dir`.

Tool calls:
1. grep for `landed as`, `landed-commit`, `empty diff` and the darn key names across `.md`, `.go` and `.yaml`: to find docs, stencils and fixtures quoting the changed outputs, which the manifest did not hold.
2. grep `markers_test.go` for darn rows: to see whether the stencil marker test needed the new fragment or marker.
3. Read the body of `markers_test.go` (lines 21-66, 100-135): to tell whether that test covers every registry entry or a chosen list.
4. grep `contracts/` for `does not license` and `report_dir`/`docs/research`: to find the review rubric that judges a darn run's scope.
5. grep `loom-rubric-code-review.md` for the scope bullet and markers: to confirm the darn Code-Review rubric carries no profile- or `report_dir`-specific allowance.
