# Card 9 — mark-guards

**Edit:**
- `cmd/lyx/agentfiledissues_test.go`
- `cmd/lyx/boardguard_test.go`
- `cmd/lyx/checkedcall_test.go`
- `cmd/lyx/configstrictness_test.go`
- `cmd/lyx/constraintchokepoint_test.go`
- `cmd/lyx/crosscompile_test.go`
- `cmd/lyx/cwdmutation_test.go`
- `cmd/lyx/destructiveguard_test.go`
- `cmd/lyx/ghguard_test.go`
- `cmd/lyx/gitrepoboundary_test.go`
- `cmd/lyx/hermeticenv_test.go`
- `cmd/lyx/llmtier_test.go`
- `cmd/lyx/norecoverydeny_test.go`
- `cmd/lyx/prerunlogging_test.go`
- `cmd/lyx/rawgitmutation_test.go`
- `cmd/lyx/registration_test.go`
- `cmd/lyx/sandbox_coverage_test.go`
- `cmd/lyx/spawnobservability_test.go`
- `cmd/lyx/spawnrole_test.go`
- `cmd/lyx/tierpurity_test.go`
- `cmd/lyx/tmuxisolation_test.go`
- `cmd/lyx/uncontainedwrite_test.go`
- `cmd/lyx/verifiedtree_test.go`
- `internal/battenrecipe/seam_enforcement_test.go`
- `internal/battenshed/seam_enforcement_test.go`
- `internal/cliwire/bannedecl_enforcement_test.go`
- `internal/cliwire/callerset_enforcement_test.go`
- `internal/cliwire/seam_enforcement_test.go`
- `internal/configreg/registration_test.go`
- `internal/fabricengine/mergevocab_test.go`
- `internal/gitkit/callerset_enforcement_test.go`
- `internal/gitrepo/noforceadd_test.go`
- `internal/loomcli/bootstrap_test.go`
- `internal/loomrecipe/seam_enforcement_test.go`
- `internal/lyxcwd/docslink_test.go`
- `internal/lyxcwd/enforcement_test.go`
- `internal/lyxcwd/raddle_guard_test.go`
- `internal/orchengine/seam_enforcement_test.go`
- `internal/pairteardown/teardown_enforcement_test.go`
- `internal/planglyph/chokepoint_enforcement_test.go`
- `internal/planglyph/status_enforcement_test.go`
- `internal/planparser/shape_enforcement_test.go`
- `internal/planparser/shape_test.go`
- `internal/reedengine/panebin_enforcement_test.go`
- `internal/reedengine/selvagepane_enforcement_test.go`
- `internal/shedbuild/seam_enforcement_test.go`
- `internal/shedcli/table_test.go`
- `internal/shedrecipe/seam_enforcement_test.go`
- `internal/shedverbs/seam_enforcement_test.go`
- `internal/shuttleengine/completionsignal_enforcement_test.go`
- `internal/shuttleengine/seam_enforcement_test.go`
- `internal/testkit/enforcement_test.go`
- `tools/sandbox/pathresolve_guard_test.go`
- `contracts/stencils/claims_test.go`
- `contracts/stencils/citation_enforcement_test.go`
- `pattern/PATTERN-test-speed.md`

**Intent:**
Decision D5: every existing test that scans the repository rather than exercising its imports carries `//lyx:guard` on the line directly above its `func Test…` line, the way `//testtiming:keep` sits above a kept test.
The round gate (card 10) runs every marked test by name whatever the impacted set, because a guard's result depends on files no import edge names.

The targets are the test files that walk the repository through `scankit` plus the two stencil scans the decision names.
In each, the marker goes on every top-level test that reads repository files outside its own package directory: the `TestEnforcement_*` scans in `lyxcwd`, `contracts/stencils/claims_test.go`, the tier-purity, LLM-tier and markdown link-integrity guards, and the enforcing tests of `PATTERN.md`'s `(test)` entries.
A test in a target file that reads only its own package directory is left unmarked, since the impacted set already selects it whenever that package changes.
No marked test sits in a `tmux` or `llm` file; card 8's derivation refuses one.

`pattern/PATTERN-test-speed.md`'s background states the marker: what it means, where it sits, that a scanning test is one that reads repository files outside its own package directory, and that whether a scanning test carries it is review discipline.
Bound, stated there too: an unmarked scanning test is not run by the round gate, so a violation passes that round and is caught by the full plan verify at Publish or Finalize, never on `main`.

**ImpactSummary:** Adds a comment line above each repository-scanning test; no test body or production code changes.
