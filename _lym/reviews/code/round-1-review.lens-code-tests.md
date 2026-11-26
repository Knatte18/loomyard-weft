```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens. For each behaviour the diff adds or changes, I name the test row that would fail if it broke, or none.

**Behaviour → pinning test**

- **`gatefixer.LastTurnMessage`:**
  - Newest stop's message comes back trimmed: `TestLastTurnMessage` "the newest stop's message is trimmed".
  - A stop before the offset is skipped: `TestLastTurnMessage` row 2.
  - No stop, a message-less stop, and a missing file each give "": rows 3–5.
  - Bytes from the offset onward are parsed, and an offset past the end is clamped: none (mutation survivor at `strand.go:43`).
- **`awaitTerminal`, dead/asking with a message:** becomes waiting with a one-line need, uncounted, not terminal. Pinned by `TestRecoverBatch_SecondCall` "a turn end carrying a message is recorded waiting with the message as its need".
- **`awaitTerminal`, dead/asking with no message:** stays dead/asking. Pinned by the existing `checkRecoveryAsking` rows of `TestRecoverBatch_SecondCall`.
- **`awaitTerminal`, `LastTurnMessage` error propagation:** none.
- **`askingNeed`, whitespace joining:** pinned by the same SecondCall row (exact `wantNeed`).
- **`Rebaseline` on a terminal failed, dead or stuck card:** the card is recorded in AmendedCards as unrendered and left out of CardsAmended. Pinned by the three `TestRebaseline_EditedPlan` rows through `checkTerminalCardAmended`.
- **`recoveryRetryRead` with an unrendered amendment at the cap:** retries. Pinned by `TestRecoveryRetry` "a dead recovery at two recoveries holding an unrendered amendment retries".
- **`recoveryRetryRead` with only a rendered amendment:** no retry. Pinned by the matching row.
- **`existingReportRemedy` follow-on:** names recover-batch once more. Pinned by the changed `TestBeginBatch_Refusals` row.
- **`runplan.RecordedOverviewFrame`, moved from websterengine:**
  - Frame, no hash and absent copy: `TestNewPlanGate` frame rows and the `TestRebaseline_*` overview rows.
  - The no-parseable-Card-Index cause: `rebaseline_test.go:900`.
- **`overviewFrameEditedFindings` / `unframedOverviewFindings`:**
  - Frame from the baseline copy fails on an edit outside the Card Index and allows a Card Index edit: two `TestNewPlanGate` rows.
  - Missing copy fails on any edit and passes an unedited overview: three rows.
  - No overview hash raises nothing: `RecordWithoutAnOverviewHashReportsNothing`.
  - An unreadable overview is an error: the existing subtest, now passed a runDir.
  - The gate's no-parseable-Card-Index cause: not covered by its own row, but it takes the same branch as the absent copy, so I raise no finding.
- **`statusFindings` / `createdHandleGlyphs` / `createdBy`:**
  - Use after the creating card passes: `TestValidateFormat_MissingMemberFindings` "UseAfterCreateResolves".
  - Use before the creating card names the creator: "UseBeforeCreateNamesCreator".
  - A card citing the member it creates itself (`creator == c.Number`): none.
  - A forthcoming card's Create handle (maps to 0) passing a pending use: none.
  - A match through `r.ID` when the spelling differs from the handle body: none.
- **`missingMemberSiblings` / `siblingMembers`:**
  - Owner-type members are listed: "MissingMemberOfTypeListsItsMembers".
  - Owner-less package members are listed: "MissingFuncOfPackageListsOwnerlessMembers".
  - An owner that is not a type (`NotATypeError`) lists nothing: none.
  - Go external-test `&lt;dir&gt;_test` unit listing from its package directory (fix commit 4cede865): none.
  - Created target skipped (no query): indirectly covered by "UseBeforeCreateNamesCreator".
- **`missingDetail` "no card of the plan creates it":** pinned by the two member-listing rows.
- **`planparser.GoPackageDir` export:** rename only, covered by the existing `TestGoPackageCounterpart` and `TestCardTargetDirs`.
- **`TargetHolds` survivor at `targetdirs.go:307`:** rename-only line, no behaviour change, no finding.

**Findings**

1. **MEDIUM · design** · `internal/planglyph/repo.go` `siblingMembers`, the `GoPackageDir` fallback
   - Bug fix commit 4cede865 adds no regression row.
   - No test lists the members beside a missing member of a `&lt;dir&gt;_test` unit, and before the fix that case failed as quarry unavailable.
   - `TestValidateFormat_MissingMemberFindings` uses only `sub/a.go`.
   - Evidence: `repo.go:254-257`; `planglyph_test.go:144-200`.
2. **LOW · design** · `internal/gatefixer/strand_test.go` `TestLastTurnMessage`
   - Mutation survivor at `strand.go:43`: no row checks that the bytes after the offset are what gets parsed, and no row has an offset past the file's end.
   - Every fake ignores `data` except row 2, which wants "" anyway, so negating the clamp survives.
   - Missing check: a fake that returns a stop with a message only when `data` holds the post-offset bytes, plus a row with an offset beyond the file's length.
   - Evidence: `strand.go:43-45`; `strand_test.go:84-131`.
3. **LOW · design** · `internal/planglyph/resolve.go:309`, `creator &lt;= c.Number`
   - The equality boundary is uncovered: no row has a card cite, as Edit or Uses, the member its own Create handle names. The issue's card-5/6 case relies on that.
   - Evidence: `planglyph_test.go:161` (creator 1, use on card 2 only).
4. **LOW · design** · `internal/planglyph/create.go` `createdHandleGlyphs`, forthcoming cards mapped to 0
   - No row has a pending card use a member that a forthcoming (in-flight) card's Create handle names.
   - Evidence: `create.go:31-33`; the only forthcoming tests (`TestResolvePass_ForthcomingCreateTargetExcludedFromStatus`) cover the Create target itself.
5. **LOW · design** · `internal/planglyph/resolve.go` `createdBy`, the `r.ID` match
   - No row has the target's spelling differ from the handle body while its parsed ID matches.
   - Evidence: `resolve.go:327-333`.
6. **LOW · design** · `internal/planglyph/repo.go:239-242`, the `NotATypeError` branch
   - No row covers a missing `Owner.Member` whose owner is a declaration that is not a type (e.g. `sub#Foo.X` with `Foo` a func), which should give a finding with no member list rather than an error.

Tool calls:
1. `git diff --stat` over the test files plus a grep of the planglyph tests for `_test#`, `NotAType` and created-by rows, and of the tests for `RecordedOverviewFrame` and the no-Card-Index cause. My context did not show which branches had tests beyond the diff pieces.
