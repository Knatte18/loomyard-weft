```yaml
lens: code-scope
outcome: reported
```

Task: trace each change in the diff to a goal point of the board entry (code-scope lens).

**Decisions.** The answer key is the decision record alone. It has no `Added after Discussion` entries, so the decisions are the two goal points under `## Goal`:
- **G1:** the darn writer stencil for profile `docs` requires the report as a committed file under `docs/research/`.
- **G2:** Land refuses an empty code diff against the parent, or at least flags it in dobby's line.

**Per change:**
- **C1 — the `report_dir` darn key → G1.** Covers the key, its `CategoryNone` class and the `LoadConfig` check that keeps it inside the worktree (`internal/darnengine/config.go`). It also covers the `template.yaml` row, the `doc.go` paragraph and the `config_test.go` rows. The `internal/configreg/configreg_test.go` expected-key list is forced by the new key. G1 is built as a config key, not a docs literal in code, which `PATTERN-code-conventions` ("No hard-coded choice") favours.
- **C2 — the report passage → G1.** Covers the new `contracts/stencils/darn/darn-fragment-report.md`, its registration in `contracts/stencils/stencils.go` and the `{{.report_directive}}` marker plus header comment in `darn-template-write.md`. It also covers `reportPassage`, `reportStencil` and the `DarnTable` wiring in `internal/darnengine/spec.go`, plus the `spec_test.go` row.
- **C3 — docs profile sets `report_dir: "docs/research"` → G1.** Covers `internal/runprofile/template.yaml:92` and the `runprofile_test.go` want map and doc comment. This is what scopes G1 to the `docs` profile.
- **C4 — Finalize computes the empty-diff flag → G2.** Covers `landedDiffEmpty` and the `recordLanding` call in `internal/landingshed/finalize.go` (lines 741 and 751–765), the `Deps.RecordLanding` signature in `deps.go`, the `doc.go` paragraph and `TestFinalize_RecordsWhetherTheLandedDiffIsEmpty`. The existing tests' closure signatures in `finalize_test.go` change because the signature changed.
- **C5 — the landed-commit file carries the flag → G2.** Covers `LandedCommit`, `emptyDiffMark`, `WriteLandedCommit` and `ReadLandedCommit` in `internal/shedrun/landed.go`, plus `doc.go` and `landed_test.go`.
- **C6 — the flag reaches dobby's line → G2.** Covers `RunView.EmptyLanding` (`internal/dobbyengine/passes.go:77`), `landNoticeLine` (`landnotice.go:316–320`), the `doc.go` lines and the `landnotice_test.go` row. It also covers the `internal/battencli/dobbyreads.go:236` read and the `internal/beamcli/dobbypairanchor_test.go` row.
- **C7 — the `RecordLanding` seam in `internal/loomcli/landingdeps.go:236–241` and its integration test:** forced by C4's signature change and C5's new type.
- **C8 — the comment line-break commits (`2e5467fe`, `5089900d`, `f15a302f`):** forced. They only re-break comments this diff added.

**Per decision:**
- **G1** is built by C1, C2 and C3.
- **G2** is built by C4, C5 and C6, with C7 forced. The diff takes the "flag" branch the entry allows and adds no refusal path in Land: `recordLanding` only logs and never changes the verdict (`finalize.go:740–746`).

**Findings:** none. Every change traces to G1 or G2 or is forced by a traced change, and each goal point is built. No `Pattern-Departure:` trailer is present in `git log f06acf39..HEAD`, and none is needed.

Tool calls: none; everything I needed was already in my context.
