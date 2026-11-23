---
verdict: APPROVED
prior:
  - id: F1
    status: resolved
    answer: WriteTreeAtRevision now writes filemode.Regular, Deprecated and Executable entries and skips the rest; writeBlobFile gives Executable 0o755, and the test row asserts the owner's execute bit and an absent symlink.
  - id: F2
    status: resolved
    answer: gateslot.scoperRedFirstMode is a table keyed by scoper name, read through Config.RedFirstMode; redfirst.go branches only on the mode, with no scoper literal in the diff's new code.
  - id: F3
    status: resolved
    answer: judgeDotnetRedFirst reads the tier summary through gateslot.TierDiscovered, and an exit-0 run with none discovered sets noTestRan; run then exits through redFirstCannotJudge naming the filter terms, as the AFilterThatSelectedNoTestCannotBeJudged row pins.
  - id: F4
    status: resolved
    answer: A run that failed with no test named, no compile error under the project, no summary or none discovered, and no timeout or crash line sets noTestRan and exits unjudged; a timeout, a crash or a run that discovered tests keeps exit 1, and summary.md's What exits 1 matches.
  - id: F5
    status: resolved
    answer: The TestsOfTwoProjectsRunOncePerProjectAndBothProjectsRedTestsAreNamed row asserts one run per project with its own .csproj target and terms, and both projects' red tests in red_tests.
  - id: F6
    status: resolved
    answer: TestGateTest_CheckoutRootMovesTheBuildAndTierDirs asserts the build and both tiers ran under the checkout; it passed in this round's tier1 run.
  - id: F7
    status: resolved
    answer: The ExpressionBody_OpeningWithABracket_EndsAtSemicolon and TheFilterTermsAreSortedAndHoldOverloadsOfOneMethodOnce rows pin both contracts.
  - id: F8
    status: resolved
    answer: The mixed abstract, failed-export and deleted-file rows cover the three branches.
  - id: F9
    status: resolved
    answer: TestTierFailureNames has the row with a failure line under a build run.
  - id: F10
    status: resolved
    answer: The keep marker sits directly above the func line.
  - id: F11
    status: resolved
    answer: nearestProject returns a reason for a read error other than fs.ErrNotExist, and TestNearestProject_ClimbsPastAMissingDirectoryAndRefusesAnUnreadableOne pins both the climb and the refusal.
  - id: F12
    status: resolved
    answer: doc.go names `<package>.<Name>` under the go scoper and `<Class>.<Method>` under the dotnet scoper.
  - id: F13
    status: resolved
    answer: docs/overview.md lists a C# test file whose project cannot be told and a C# run that discovered no test.
---

Round 2 re-read the whole range against the issue #208 entry, with the round-2 focus entries driven in order.
Every round-1 finding is resolved, and no changed hunk holds a new BLOCKING defect.

I drove the real substrate: `lym gate test --tier 1 ./internal/gatecli` with `-run 'TestGateRedFirst_Dotnet|TestGateTest_CheckoutRootMovesTheBuildAndTierDirs'` on this machine, which has `dotnet`.
Both passed, with every `TestGateRedFirst_Dotnet` row running rather than skipping: the red row, the row whose test does not build on the base, and the green row.

## Focus verdicts

- **Executable blobs and links in `WriteTreeAtRevision`.**
  Not a defect.
  go-git's `FileIter.Next` already skips `filemode.Dir` and `filemode.Submodule` entries.
  The switch admits `Regular`, `Deprecated` and `Executable` and drops `Symlink`.
  `writeBlobFile` opens an `Executable` blob with 0o755.
  The doc comments of `WriteTreeAtRevision` and `ExportCodeTreeAt` state that behavior.
- **Hard-coded scoper choice.**
  Not a defect.
  The new code branches on `gateslot.RedFirstMode`, read from `scoperRedFirstMode`, a table keyed by scoper name.
  The `cfg.Scoper == "go"` in `run.go` and the validation literal in `config.go` predate the range.
  The Go path's messages, envelope keys and exit codes are unchanged:
  `redFirstNoTest` and `redFirstVerdictExit` carry the old text verbatim, and `extra` is nil on the Go path.
- **No-test and pre-test failures.**
  Not a defect.
  An exit-0 run with no test discovered, and a failed run with no named failure, no in-project compile error and no timeout or crash line, end in `RedFirstUnjudgedExit` with a named reason.
  A run that started tests and judged none keeps exit 1, matching `redFirstVerdictExit` and the exit contract in `internal/gatecli/doc.go`.
- **`nearestProject` and `byProject`.**
  Not a defect.
  A read error other than `fs.ErrNotExist` returns a reason, and the two-project row shows one run per project with its own target and terms.
- **Tests added or fixed this round.**
  Each asserts what its name states and pins behavior no other test covers.
  The redundancy report lists each as unique, kept or out of process.

## Rejected

- Projects whose shared build fails before a tier runs: the captured output holds only a build summary line, so `TierDiscovered` reports none found and the run reads as no test ran, as intended.
- A run's `checkoutRoot` lies under the system temp directory, outside any git repository, so the stray-file listing is skipped there rather than failing a green run on `bin/` and `obj/`.
- Noted, not raised, since an incremental round raises only BLOCKING findings on changed hunks:
  - `projectCompileError` counts a compile error in any file of the test project as red for every selected test.
    A changed helper file of that project that does not build on the base can therefore pass a test that would be green there.
    Narrowing the match to the selected tests' own files would tighten it.
  - The unsupported-scoper refusal still prints `noteScoper`, which reads "not go" now that dotnet is supported too.
  - `tierDiscoveredPattern` repeats `tierSummaryPattern`'s shape rather than adding a capture group to it.
