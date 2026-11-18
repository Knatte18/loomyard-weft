# Card 3 — drift-verify-loosened-checks

**Create:**
- `internal/driftengine#LoosenedCheckTrailerKey`
- `internal/driftengine#loosenedTrailerLines`
- `internal/driftengine#fileRulingKey`
- `internal/driftengine#listedFiles`
- `internal/driftengine#DriftVerify.listFiles`
- `internal/driftengine#uncausedReason`
- `internal/driftengine#loosenedLines`

**Edit:**
- `internal/driftengine#Deps`
- `internal/driftengine#DriftVerify.Call`
- `internal/driftengine#DriftVerify.checkAndJudge`
- `internal/driftengine#DriftVerify.judge`
- `internal/driftengine#DriftVerify.judgeRequest`
- `internal/driftengine#DriftVerify.storeRulings`
- `internal/driftengine#DriftVerify.passCommand`
- `internal/driftengine#DriftVerify.storeCaseRulings`
- `internal/driftengine#renderVerdict`
- `internal/driftengine#TestDriftVerify`
- `internal/driftengine#verifyFixture`
- `internal/driftengine#newVerifyFixture`
- `internal/driftengine#verifyFixture.judgeArrives`
- `internal/driftengine#TestVerdict`
- `//internal/driftengine/doc.go`
- `internal/loomcli#driftDeps`
- `internal/loomcli#loomCLI.wire`
- `internal/loomcli#TestWire_ReviewKeysReachTheEnv`
- `internal/shedrecipe#driftVerifyEntry`
- `internal/shedrecipe#driftTestEnv`
- `internal/shedrecipe#TestDriftEntries`
- `internal/testkit/envkit#driftDeps`
- `internal/testkit/envkit#nilLegal`
- `internal/battencli#intentionallyNil`

**Uses:**
- `internal/loomcli#taskMergeBase`
- `//internal/shedrecipe/doc.go`
- `internal/battencli#TestWire_EverySeamFilled`
- `internal/loomcli#TestWire_BeamRecipe`
- `internal/loomcli#TestWire_EverySeamFilled`
- `internal/loomcli#TestWire_SamplerRecipe`
- `internal/testkit/envkit#TestFullEnv_FillsEverySeam`
- `internal/testkit/envkit#TestNilSeams_ReportsExactPaths`
- `internal/testkit/envkit#TestNilSeams_SkipsNilLegalPaths`

**Intent:** D3: Drift-Verify and Drift-Recheck list each file of the review range that exists at the review base, have the judge rule its loosened checks, store the rulings, halt on an uncaused one, and the verdict lists them.

Seams, on `Deps`:

- `ReviewBase func() (string, error)`: the review base the Code-Review rubric's range starts from, the merge base with the last merged parent tip.
  Nil lists no file: a standalone run records no parent branch, so it has no review range, as its Code-Review reads `none`.
- `ChangedFiles func(base string) ([]fabricengine.FileDiff, error)` and `CommitMessages func(base string) ([]string, error)`: the files existing at `base` that the commits change, with their diffs, and the whole messages of the commits from `base` to `HEAD`.
- `loomcli`'s `driftDeps` fills `ChangedFiles` and `CommitMessages` through `fabricengine.Open(location)`, card 1's `CommittedFileDiffs` and `CommitMessages(base, "HEAD")`.
- `loomCLI.wire` sets the drift deps' `ReviewBase` to the same `verifyMergeBase` value it sets as `Env.ReviewBase`: `taskMergeBase(location)` when the origin record exists, nil otherwise.
  One value feeds both, so Drift's base is the Code-Review range's base, and a Parent-Align merge's parent commits lie below it, so a test they loosened is never listed (criterion D3).
- `driftVerifyEntry` requires `Drift.ChangedFiles` and `Drift.CommitMessages`; `Drift.ReviewBase` is nil-legal, and `envkit`'s `nilLegal` lists it while its `driftDeps` fills the other two with empty answers.
- batten wires no Drift row, so `battencli`'s `intentionallyNil` gains `Drift.ChangedFiles` and `Drift.CommitMessages` with the reason `loom- and darn-only seam, batten has no Drift row`.
  `Drift.ReviewBase` needs no entry, since `NilSeams` skips the paths `envkit`'s `nilLegal` lists.

Listing, `listFiles`, run by every pass that reaches the check (`checkAndJudge`, after its unexpected-cell and stored no-match halts, and `passCommand` before it reads the compare's exit):

- A pass that ends before the check lists no file: no scope record, scope off, a deferred or waived capture, a capture that never counted, and in `checkAndJudge` an unexpected cell or a stored no-match ruling, whose halt records a check with no `Loosened`.
  That bound is stated in `doc.go`: there D1's rubric bullet is the one check of a loosened test, and the resume after fixing a halt reaches the listing.

- It reads the base, the changed files and the messages, and collects the range's trailer lines with `loosenedTrailerLines`: every line of a message's last paragraph that opens `Loosened-Check:`, trimmed, sorted and deduplicated.
- Each file's key is `fileRulingKey`: the sha256, hex, of the file's diff, a NUL, and the trailer lines joined by newlines.
  So a commit that only adds or changes a trailer changes every key, and a file whose diff and the range's trailers are unchanged keeps its key.
- A file whose path and key a stored `JudgedFile` holds reuses that ruling; the rest are the files to judge.

The judge, in both diff modes:

- It spawns when a cell or case needs it, as today, and also when a listed file has no stored ruling, even with no changed cell and with a compare that exited 0 (criterion D3).
  The request carries the unruled files as `Files`, ids `f1`, `f2`…, and every range message as `CommitMessages`; a spawn for files alone carries no cells and, under command, `CompareLog` empty.
- `storeRulings` and `storeCaseRulings` also store each file ruling as a `JudgedFile` with its path, key, strand and time, replacing an earlier one of the same path, in the same write that records the check.
- The check's `CheckResult.Loosened` lists every loosened check of the listed files, stored or new, with its path, test, what was loosened, ruling, cause and reason.

The halt:

- Any listed file's `uncaused` check, reused or new, makes the pass a Stuck for the parent through `haltForParent`, recorded as a check that did not pass.
  When the same judge spawn also rules a cell or case `does not match`, that halt's reason comes first and the uncaused reason follows it in the one Stuck.
- `uncausedReason` words it: `a test the change loosened has no measured cause: <path>: <test>: <what was loosened>: <reason>; …; way forward: commit a "Loosened-Check: <test> — <what was loosened> — <measured cause>" trailer on any commit of the range, with no change if need be, or commit a revert of the loosening, then lym loom resume`.
- A `caused` or `none` ruling for every listed file lets the pass end as the cells and cases decide.
- Bound: a trailer's measured cause is the committer's attestation, which neither the judge nor the code re-measures.
  Its bound is the verdict's `Loosened checks` lines, which carry each check with its ruling and cause into the pull request's body and PR-Review's `drift_verdict`.

The flow the criteria name: a widened limit in a test existing at the base, with no trailer, spawns the judge and halts on its `uncaused` ruling;
a following commit adding only the trailer changes the file's key, so `lym loom resume` spawns the judge again, which rules `caused`, and the pass is Done.
Drift-Recheck over unchanged diffs and trailers reuses every stored file ruling and spawns no judge for them.

The verdict: `renderVerdict` adds `loosenedLines(scope.LastCheck.Loosened)` after the check lines: `- Loosened checks:` then one line per check, `` `<test>` in `<path>`: <what was loosened>; <ruling>; cause: <cause or none> ``.
The pull request's body and PR-Review's `drift_verdict` carry it, since both render `Verdict`.

`doc.go` gains the listing, the file rulings and their key, the uncaused halt and its way forward; the rows run only where `drift.enabled` turns them on, and the writers' drift gates stay without the judge.

Tests, tier0, all rows of the existing tests over their fakes.
`verifyFixture` gains `changedFiles`, `commitMessages` and `reviewBase` answers that `newVerifyFixture` wires into `Deps.ChangedFiles`, `Deps.CommitMessages` and `Deps.ReviewBase`, and records the base each read was called with.
`verifyFixture.judgeArrives` writes the request's file rulings from a new per-row `fileRulings` field, beside its cell and case rulings.
A trailer-only commit is the fixture's `moveHead` with a changed `commitMessages` answer.

- `TestDriftVerify`, lines mode: a widened limit with no trailer and no changed cell spawns one judge whose request lists the file and the messages, and its `uncaused` ruling halts with the way forward above;
  the base `ChangedFiles` and `CommitMessages` were called with is `ReviewBase`'s answer, not `BranchPoint`'s;
  a resume with no new commit spawns no judge and halts again on the stored `uncaused` ruling;
  then the same file with a trailer-only commit added spawns the judge again, which rules `caused`, and the pass is Done;
  the next pass over unchanged diffs and trailers spawns no judge and reuses the ruling;
  a judge spawn ruling a cell `does not match` and a file `uncaused` halts with both reasons, the no-match one first;
  a nil `ReviewBase` lists no file and spawns no judge.
- `TestDriftVerify`, command mode: a compare that exits 0 with an unruled listed file spawns the judge for the file alone, and its `uncaused` ruling halts with the way forward above in place of the Done a passing compare gives today.
- `TestVerdict`: a row whose last check holds a caused and an uncaused check renders the `- Loosened checks:` lines exactly.
- `TestDriftEntries`: shedrecipe's `driftTestEnv` fills `Drift.ChangedFiles` and `Drift.CommitMessages` with empty answers, so `VerifyBuilds` still builds;
  new rows `VerifyNilChangedFiles` and `VerifyNilCommitMessages`, beside `VerifyNilCompare`, refuse naming `Drift.ChangedFiles` and `Drift.CommitMessages`.
- `TestWire_ReviewKeysReachTheEnv`: on a pair run the drift deps' `ReviewBase` answers the same commit as `Env.ReviewBase`.

**ImpactSummary:** Drift-Verify and Drift-Recheck gain a file-listing step, the judge's file rulings and an uncaused halt; three new Deps seams wired in loomcli, required by the DriftVerify registry row and filled by envkit.

**Commit:** `3: drift-verify-loosened-checks`
