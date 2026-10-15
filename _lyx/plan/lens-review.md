# Lens review

Every lens fork returned a report.

- criteria-coverage, MEDIUM, spec row, preamble and `Glyph.UnitPath` sentences have no test: disputed, every fix worse. No test harness reads spec or `doc.go` prose (`contracts/specs/specs_test.go:17-74` holds only `TestRegistry_*`), so pinning it means a new prose wording test, and the record's Out list routes wording tests toward removal (`lighter-test-rules`); a `doc.go` comment has no test route beyond `lyx loom lint-comments`. The criteria stay prose checked at review.
- criteria-coverage, LOW, the way-forward row pins only two of D5's clauses: fixed as proposed; the `Edit` row in `TestCheckEditTestFile` compares the whole `ValidationError`, `Detail` included.
- criteria-coverage, LOW, the old Create-rule wording and "a test file the card adds tests to" are not pinned gone: fixed as proposed; card 3's `wantNone` claim adds both.
- criteria-coverage, LOW, the rework gate is pinned only at the index: fixed as proposed; card 1 adds an `EditTestFile` row to `TestGateParity_ReworkPlanGate` through a re-signed `reworkParityFixture` with an `edit` parameter, its other caller listed.
- branch-coverage, MEDIUM, the ledger skip of a path- or handle-shaped `Edit` ref is unreached: fixed as proposed; rows for a path-shaped `internal/x/foo_test.go` and a `plan:` handle under `Edit`, each with no finding.
- branch-coverage, LOW, the parse-error skip is unreached: fixed as proposed; a row with a glyph-shaped `Edit` ref that fails `glyph.Parse`.
- branch-coverage, LOW, the not-ok `UnitPath` skip has no reaching input: fixed as proposed by dropping it; card 1 states no branch reads the ok result, since quarry's `Glyph.UnitPath` always answers ok for a Go glyph (`quarry@v0.2.1/glyph/unitpath.go:19-26`).
- branch-coverage, LOW, no row has an ignored name only in the base name: fixed as proposed; row `internal/x/_foo_test.go#`.
- test-economy, LOW, the single-card done row repeats the two-card done row: fixed as proposed; the single-card row is dropped.
- test-economy, LOW, the index done row reads as re-testing the skip: fixed as proposed; card 1 says it pins that `index.ValidateFormat` forwards its `done` set.
- test-economy, LOW, the production `.go` and `.md` rows reach one branch: disputed, every fix worse. The record's acceptance criteria name a production `.go` path and an `.md` path as separate inputs (`_lyx/discussion/decision-record.md`, Acceptance criteria, third bullet); merging leaves one of them unpinned.
- row-reachability, BLOCKING, the index rows run on `minimalPlan`'s `language: none` plan, where the check reports nothing: fixed as proposed; `TestIndex_AppendsPlanChecks` gains a per-row `language` field and the `edit-test-file` rows run under `go`.
- row-reachability, LOW, the parity row's `sub/a_test.go` content is unnamed: fixed as proposed; card 1 names a `package sub` file with an empty `TestA`.
- forced-edits, MEDIUM, `caller-uncovered`'s way forward offers a test file the new check refuses: fixed as proposed after the parent recorded it (decision "Gate ways forward and Plan-Review rubric follow the test-file refusal"); new card 2 re-signs `coverageVerbAndWay`, updates spec row 35 and pins both branches in `TestGlyphChain_CallerCoverage`.
- forced-edits, LOW, `redundant-file-target`'s way forward has the same conflict: fixed as proposed under the same parent decision; card 2 changes `planGatePass`'s detail for a `_test.go` file, spec row 33, and adds a `TestGlyphChain_RedundantFile` row.
- forced-edits, LOW, the Plan-Review rubric still asks reviewers to flag what the gate now blocks: fixed as proposed under the same parent decision; card 3 drops the sentence and turns its claim into a `mustNot`.
- forced-edits, LOW, `TestGateParity_PlanGate`'s doc comment misses the new row: fixed as proposed; card 1 names the row in it.
- forced-edits, LOW, the `NewIndex` neighbours sentence names no member: fixed as proposed; the sentence is dropped.
