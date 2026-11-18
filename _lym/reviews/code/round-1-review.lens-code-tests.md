```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, find the test row that fails if the behaviour breaks.

**Per behaviour, the test that pins it**

- **The `ending` type replaces the four-value returns across Finalize, PR-Review and Parent-Align, with the same verdicts, reasons and errors.** Pinned by the package's existing Finalize, PR-Review and Parent-Align suites. I did not trace every converted path to its own row.
- **`prreview_wait_test.go:936` now reads `.err` from `wait`.** This is a signature change only, and the test still asserts `context.Canceled`.
- **The merge-in-required retry loop in `landUnderLock`, with a second required error ending Stuck.** Pinned by `TestFinalize_MergeInRequiredRetry` (`finalize_test.go:490`), rows "retry lands", "retry also required stays Stuck" and "a code-side parent commit between the passes…". Also by `TestFinalizeVerify_RetryMergeInIsBracketedAgain` (`finalize_verify_test.go:210`).
- **The parent-holds read happens once per pass, and a failed read halts as an owner check.** Pinned by the `finalize_owner_test.go:88` row "a failed read of whether the parent holds the work halts". A held parent skipping the land check is pinned at `finalize_land_test.go:79`/`:91`. The sync skip is covered by `finalize_test.go:829`.
- **Removal of the land check's own halt on a failed held read.** Covered by the owner row above, which fires first.
- **`landingSummary`: a missing or malformed description, unreadable engines or an unreadable regression check is an error before any commit or merge.** Pinned by `TestFinalize_UnusableMessageInput_ErrorBeforeMergeOrCommit` (`finalize_test.go:423`). The new error text "read the change description" is not asserted (finding 3).
- **`checkWiring` by reflection: each required field named when unset.** Pinned by `prreview_test.go:559` "NilSeamIsAnErrorNamingIt", which covers every required field and `TaskHead`.
- **`checkWiring`'s optional list (`ParentReviewWanted`, `AskParent`, `BeamRecord`, `DropBeamRecord`).** `BeamRecord` and `DropBeamRecord` are left nil by the PR-Review fixture, so their rows pin them. No row pins `ParentReviewWanted` or `AskParent`: the fixture sets both (`prreview_test.go:780-781`, `:803`), and no test sets either to nil (finding 1).
- **`parseVerdictReport` and `bullets`, shared by both report parsers.** Pinned by `TestParseFixerReport` (`fixerreport_test.go:17`) and `TestParseFixReviewReport` (`fixreview_test.go:17`).
- **`soloSession.table` and `requireValues` for the Describe, Fixer and fix-review tables.** A missing describe stencil is pinned at `describe_test.go:302`; an empty required input at `describe_test.go:278`. Seat parity is pinned by shuttleparity's `TestSeats_CodexManifestHoldsClaudes` and the listed-holder scan `TestLaunchSpecLiterals_AreListed`, which require `seats.go`'s new holders.
- **`readRowAttempts` at Parent-Align: the scratch record moves first, then the row entry is read.** The scratch move is pinned at `parentalign_fixer_test.go:541`. A nil `RowEntry` reading as entry 0 at Parent-Align, which the summary names as new, has no row: the fixture always wires `RowEntry` (`parentalign_test.go:149`) (finding 2).
- **`readMergeBefore`, shared, with the earlier record unreadable read as absent.** Covered by the existing Finalize and Parent-Align merge-record tests through their fixtures. I did not locate a row with an unreadable earlier record; the only change there is log text, so I raise nothing.
- **`startFailure`, `gateCall`, `runMarked`, `pushTask`, `closeIfOpen`, `findOrOpenPullRequest`, `commitPRRecord`/`removePRRecord` and `githubFailure`.** Exercised by the existing fixer, verify, Finalize-PR and PR-Review scripted-call suites. I did not open each row within the call budget, and nothing I read shows a gap.

**Findings**

1. **LOW, class `design`.** Location: `internal/landingshed/prreview.go`, `optionalPullRequestSeams` and `checkWiring`.
   - The diff adds a name-keyed optional-seam list that reflection reads, and no test runs PR-Review with `ParentReviewWanted` or `AskParent` nil.
   - Dropping either name from the list, or renaming the field, makes the row refuse runs that leave it unset, and no row would fail.
   - Evidence: the fixture sets both at `internal/landingshed/prreview_test.go:780-781`, and grep finds no `ParentReviewWanted = nil` or `AskParent = nil` in `prreview*_test.go`.
   - Fix: add a row to `NilSeamIsAnErrorNamingIt`'s test, or a sibling row, that leaves each optional seam unset and asserts the call does not fail on wiring.
2. **LOW, class `design`.** Location: `internal/landingshed/fixerattempts.go`, `readRowAttempts`, as `ParentAlign.readAttempts` now calls it.
   - Parent-Align now reads a nil `RowEntry` as entry 0, where before it would have dereferenced the nil seam. The summary names this under "Behaviour notes".
   - `NewParentAlign` does not require `RowEntry`, yet no Parent-Align row leaves it nil.
   - Evidence: `internal/landingshed/parentalign_test.go:149` always wires `RowEntry`, and the `NewParentAlign` nil checks at `parentalign.go:92-106` omit it.
   - Fix: add a Parent-Align row with `RowEntry` nil that asserts entry 0's record is read.
3. **NIT, class `consistency`.** Location: `internal/landingshed/finalize_pr.go`, `landingSummary`.
   - Finalize's missing-description error changed from "parse change description" to "read the change description", and no row asserts the text.
   - Evidence: `internal/landingshed/finalize_test.go:454-457` checks only `err != nil`.
   - Fix: none needed unless the text is meant as a contract. If it is, assert it in the "missing" and "malformed" rows.

**Tool calls**

1. grep of `*_test.go` for the description error, wiring, merge-in-required, `ParentHoldsTaskWork` and `RowEntry` rows: the manifest held no test files.
2. Read `prreview_test.go:550-600`, `finalize_test.go:490-560` and grep of the held-error rows: needed the bodies of the wiring and retry tests to confirm what they assert.
3. grep for the fixture's optional seams, the description-path rows, the Parent-Align scratch test and the parser tests: needed to find which behaviours have rows.
4. Read `finalize_test.go:420-460`, grep for nil optional seams, and read `parentalign_fixer_test.go:540-560`: needed to confirm the description test's assertions, that no test leaves the optional seams nil, and the scratch-move row.
