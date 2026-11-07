```yaml
lens: code-tests
outcome: reported
```

Task: for each behaviour the diff adds or changes, name the test row that pins it, or none. The review range is `f7465bd8..HEAD`. I made no tool calls and judged from the inherited context.

**Per behaviour: the test that pins it**

| Behaviour | Test | Verdict |
|---|---|---|
| `{meta_dir}` renders in a capture command and the file lands outside `{baseline_dir}` | `TestCaptureCurrent` and `TestRunCapture`, the "meta dir" subtests (`capture_test.go`) | pinned |
| A capture that wrote only a meta file does not pass | the "exit 0 with only a meta file" rows in `TestCaptureCurrent` and `TestRunCapture` | pinned |
| A backup moves the baseline's `.meta` to the backup's `.meta` | "a backup moves the baseline's meta dir…" in `TestRunCapture` | pinned |
| A missing meta dir at backup is tolerated | the existing backup tests, which have no meta dir | pinned implicitly |
| `{meta_dir}` renders in the compare command (`compare.go:88`) | none | **no row** |
| `ParseExpectation` checks marks: blank file or key, invalid glob, no matching case or key, `keys` refused under command | the `Metadata*` and `CommandMetadata*` rows in `TestParseExpectation` | pinned |
| `Merge` joins marks without duplicates | "MergeJoinsMetadataMarksWithoutDuplicates" | pinned |
| The seal writes the marks into the scope record | "TheSealRecordsTheMetadataMarksInTheScopeRecord" | pinned |
| `Check` drops marked files and keys from the scope record's marks and from the sealed expectation's marks | the two `leavesNoCell` rows in `TestCheck` | pinned |
| `Check` drops marks that come from a `lym drift amend --entries` amendment | none; `sealedAndAmended` plus `Metadata.union` in `Check` is exercised only through `Merge` | **no row** |
| Under `diff: command`, a judge's ruling on a marked case counts as `matches`, and a stored ruling on one is no Stuck (`verify.go:879` and `verify.go:902`) | none; `verify_test.go` is unchanged | **no row** |
| The gate marks a cell unchangeable by the commit rule and ends Terminal | the "baseline's capture head and the code HEAD" row in `TestDriftGateUnchangeableCells` | pinned |
| The gate marks a cell unchangeable when a second capture differs, and ends Terminal | the "differs between the gate's two captures" row | pinned |
| A mixed failure still fails without Terminal and never names the unchangeable cell | the "stable unexpected change beside a changing one" row | pinned |
| The second capture does not pass, so nothing is marked and the gate fails as before (`check.go:259`) | none | **no row** |
| A cell the second capture lacks, or one of a different `Kind`, counts as unchangeable (`check.go:280`) | none | **no row** |
| `namesCommit` accepts an abbreviation of at least 7 hex digits and rejects a shorter one | none; the commit-rule row uses a 7-digit run only | **no row** |
| `baseHead == head` disables the commit rule | none | **no row** |
| The Terminal findings name the way forward `lym drift amend … metadata:` | the `want` strings in that table | pinned |
| `Verdict` names the baseline commit and the marked files and keys | "BaselineLineAndMetadataMarksFrameTheLastCheck" | pinned |
| `Verdict` omits the baseline line when no `Drift-Baseline` commit exists | the existing verdict tests, which have no trailer, assert the old text | pinned implicitly |
| `Deps.Head` is required for the `drift` gate entry (`entries_gate.go`) | `fixture_test.go` only supplies the seam; no refusal row | **no row** |

**Findings**

1. MEDIUM, `design`. `lym drift amend --entries` with `metadata` marks, the way forward the Terminal stop names, has no end-to-end test. The parse and merge steps are tested separately.
   - Location: `internal/driftengine/amend.go:99-121`, `check.go:243-258`, `check.go:190`.
   - Evidence: no changed test in `amend_test.go` or `check_test.go` drives an amendment that holds `metadata` and expects a cell-free `Check`.
2. MEDIUM, `design`. The `diff: command` ruling change has no test, and the landing summary claims it.
   - Location: `internal/driftengine/verify.go:879` (`storedCaseRulings`) and `verify.go:902` (`storeCaseRulings`).
   - Evidence: no changed row in `verify_test.go`; the diff shows no such file.
3. LOW, `design`. The gate's second-capture branches are only partly covered.
   - Location: `check.go:259-261` (a second capture that does not pass), `check.go:279-281` (the found and `Kind` conditions), `check.go:245` (`baseHead != head`).
   - Evidence: `TestDriftGateUnchangeableCells` holds three rows and none reaches these branches.
4. LOW, `design`. The 7-digit minimum and the hex-run split in `namesCommit` have no boundary row.
   - Location: `check.go:288-300`.
   - Evidence: the only commit-rule row uses exactly 7-digit abbreviations.
5. LOW, `design`. `{meta_dir}` in the compare command has no row.
   - Location: `compare.go:88` and `steprun.go:325`.
   - Evidence: `compare_test.go` is not in the diff. An omitted `MetaDir` would fail the existing compare tests at `MkdirAll("")`, but a wrong rendering would pass.
6. LOW, `design`. The new `Drift.Head` requirement has no refusal row.
   - Location: `shedrecipe/entries_gate.go:282`.
   - Evidence: the diff changes only `fixture_test.go`, to supply the seam.

**Test economy:** I found no duplicate test and no assertion on static prose. The `capture_test.go` rows that repeat across `TestCaptureCurrent` and `TestRunCapture` exercise different entry points.

Tool calls: none.
