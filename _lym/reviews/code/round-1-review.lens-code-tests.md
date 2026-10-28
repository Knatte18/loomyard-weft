```yaml
lens: code-tests
outcome: reported
```

Lens: code-tests. Below, each behaviour is paired with the test that pins it, or `none`.

**Pinned behaviours**
- D1 and D2 id and acceptance rules: `TestCheckDecisionIDs` (`internal/discussionparser/validate_test.go:499-536`). Rows cover a malformed heading, an empty title, an added entry, a repeated id, an id in no bullet, `D1` against `D10`, a range, prose outside a bullet, a fenced heading, no acceptance section and no decisions section.
- D1 and D2 in the callers:
  - Darn caller: the "DecisionIDFindingsFollowTheSectionFindings" subtest and the table at `gates_test.go:272-273` pin the ordering against `Validate`'s findings and the skipped history.
  - Loom caller: `gates_test.go:207-219` (malformed heading beside a section finding, `Validate`'s findings first).
- `lym loom decision add` stays on `Validate` alone: `append_test.go` row "a decision heading without an id is not refused".
- D3 and D4 citations: `TestValidateLoomDiscussion_Citations` (`citations_integration_test.go:72`). It covers missing and existing paths, both marks, a placeholder, a fenced block, a directory, a trailing slash on a file, glyph and qualified symbols (including `Do()` and `Thing.Field`), both ambiguity kinds, untested non-citations, and the three skipped regions.
  - Its subtests cover an empty index, a staged root entry, a quarry outage (error wrapping `ErrQuarryUnavailable`, nil findings) and a darn record.
  - The extraction itself is pinned by `TestCitations` (`validate_test.go`).
- `ResolveGlyphs` and its `glyph-not-found` policy: `TestResolveGlyphs` (`planglyph/index_test.go`). It covers found, multipart files, ambiguous candidates and the rejected and missing rows. The per-card policy is unchanged, so the existing resolve tests guard it.
- `TrackedRootEntries`: `TestTrackedRootEntries`.
- D5 marker parser: `TestFindSeamTests` and the stacking rows in `guards_test.go`, covering both orders, misplaced and malformed markers, and the `_test` unit.
- D5 scan (unmarked `NumField`, `VisibleFields`, helper, `NilSeams` in all three spellings, non-struct marker, out-of-module marker): the fixtures in `TestSeamMarkers_ReflectiveTestsAreMarked` and its real-tree subtest.
- D6 (`seam-test-unlisted`): the `SeamTest…` rows of `TestValidatePlan_CardChecks`, including counterpart Uses and package-counterpart Uses, an edit of the test, a done card, a csharp plan and a nil reader. The "SeamTestMarkerTheParserRejectsIsAReturnedError" subtest covers the scan-error halt.
- D7 (`test-tier-unstated`): the `TestValidatePlan_CardChecks` rows, including `SameTagOnTwoLinesPasses` (the parent's added decision), `||` joins, `untagged`, two files on one line, same base name in two directories, nil tags, csharp and done.
- D8 (`doc-go-unlisted`): the `DocGo…` rows (Uses, Delete, test-only handles, done card, csharp plan).
- D9 wiring:
  - Plan-Write gate: `entries_planwrite_test.go` pins tier and seam.
  - `validate-plan` modes: `parity_test.go` pins tier and seam, in the no-flag and lens-review modes.
  - Overlay: `overlay_test.go` pins tier.
  - Rework: `validate_test.go` pins tier.
  - Gate wiring: `entries_gate_test.go` pins the new nil `PlanIndex` and relative `WorktreeRoot` refusals for the discussion gate.

**Findings**
1. **MEDIUM, class design.** `doc-go-unlisted` has no row at any D9 wiring site: the Plan-Write gate, the `validate-plan` modes in `parity_test.go`, `--rework` and `--overlay`.
   - The only coverage is the direct `ValidatePlan` call in `TestValidatePlan_CardChecks`.
   - `grep 'doc-go-unlisted|DocGo'` over `internal/loomcli/*_test.go` and `internal/shedrecipe/*_test.go` found nothing.
   - So dropping `docGoFindings` from one builder or mode would pass. Acceptance D9 names every mode.
2. **LOW, class design.** `seam-test-unlisted` has no row for `--rework` or `--overlay`. `parity_test.go` covers only the no-flag and lens-review modes, and the rework test (`validate_test.go`) and the overlay test assert `test-tier-unstated` only.
3. **LOW, class design.** `PlanContext.SeamTests` and `TierTags` are filled by three builders: `loomcli/wiring.go`, `validate.go`'s `validatePlanCmd` and `shedrecipe.planContextOf`. A `TierTags` regression in `loomcli/wiring.go` (the gate builder) has a pinning test only through the shedrecipe `Env`. No test drives `loomCLI.wire` to check that it fills `Env.TierTags` from `gate.yaml`, and the `validate-plan` builder is covered only through the parity fixtures. This is a coverage gap in the builder, not a missing behaviour.
4. **LOW, class design.** In D8's classification, the Rename new-side-with-test-old-side rule and the ambiguous-answer-is-non-test rule are not visible among the `DocGo…` rows I could list (row names only partly shown). I could not confirm a row for either. The handler or the code-correctness lens should verify.
5. **NIT, class design.** The D4 quarry-outage case has coverage in `citations_integration_test.go` (the loom function) but not at the gate (`NewDiscussionGate`) level, where the error must be returned rather than turned into a finding. `gates_test.go:416` covers the outage only for the plan gate. The gate closure returns the error verbatim, so the risk is small.

No test-economy duplication stood out. The `TestFullEnv_*` markers are required by the scan, not redundant tests.

**Tool calls (4 of 6)**
1. A `git diff` grep of test names across the new and changed test files. My context had `envkit_test.go` but none of the other test files.
2. A diff and grep of `parity_test.go`, `overlay_test.go`, `validate_test.go`, `entries_gate_test.go`, plus a search for doc-go rows. This was to check D9 mode coverage.
3. A grep for the decision-check names across the tests and for `ValidateLoomDiscussion` call sites. This was to find where D1 and D2 are pinned for loom.
4. A `git diff --stat` of `decision_test.go` and `append_test.go`. This was to check that `decision add` is unchanged on records without ids.
