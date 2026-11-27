---
dispositions:
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
  - id: F7
    disposition: fixed
  - id: F8
    disposition: fixed
  - id: F9
    disposition: fixed
---

# Code review round 1, fixer report

## Changes

- **F2** (`9845a07eb`): the transport row of `TestRunRule`, now `TransportReadAndRereadFailuresAreRecordedAndReachTheRule`, goes on to an action list whose re-read of `FieldOriginRefs` fails as transport. It wants the failure recorded under `PhaseOrigin` and the rule asked again.
- **F3** (`9845a07eb`): new row `ReadErrorMixingATransportAndAnotherFailureIsReturned`.
- **F4** (`9845a07eb`): new subtest `AFailingStoreStopsNoOther` in `TestSnapshotReader_ReadsAHub`.
- **F5** (`9845a07eb`): `RecordsStoreWhoseLayoutCannotBeRead` first reads the present store's literal `Heads` and `Tags`, and an absent directory.
- **F7** (`536d89d9d`, causes of two lines corrected in `eedb8989b`): an empty commit carrying `Loosened-Check:` trailers for the tests the finding names and for the trimmed tests with no holder in another file.
  It also records `TestClassifyPaths_PartitionsInputWithNothingLostOrDuplicated`, deleted in an earlier commit with its holder in the same file.
  The trimmed `TestMergeVocabulary_ErrorsAreSideFree`, `TestMergeState_Record` and `TestTeardownLock_SerialisesAddDiscardAndPairPush` get no new line, since tests in other files hold what they lost.
- **F8** (`66006af53`): the layout comments in `mutation.go` and `verbresults.go` no longer name engine-private functions, the engine file or engine error values.
- **F9** (`9845a07eb`, `9ca75a9a3`): `LocksPrecedeTheirReadsAndReleaseInReverse` asks for a read phase again, the F3 row passes a nil lock, `TestSnapshotReader_RefStorage` tags a blob both ways, and new subtest `AnchorsSkipAPrunableRecordsWorktree`.

- **Comment wrapping** (`e1a22edee`): a gate finding on a wrapped comment line above `TestSnapshotReader_ReadsAHub`, which F4's change added.
  The comment is rewritten as one sentence per line.

`lym gate test` passed at tier 0 and tier 1 on `internal/fabricengine` and `internal/fabriclayout`; the last tier 1 run, after `9ca75a9a3`'s change, covered `internal/fabricengine`.
`e1a22edee` changes a comment alone; after it, `go vet -tags integration ./internal/fabricengine` passed and no test was run.
No new case was run against its defect put back into the code.
The fabric `_test.go` total stays below the base total of 58594 lines.

One thing F7 surfaced and did not change: no test reads the told tip back from a conflicted `MergeInAt`'s record.
The `TestMergeInAt_MergesTheGivenTipNotTheLocalOne` trailer says so.

## Self-check

- fix-code-correctness, LOW: the annotated-tag arm of the git ref filter had no failing test. Fixed as proposed (`9ca75a9a3`).
- fix-consistency: no findings.
- fix-proposal, LOW: the `TestNew_HappyPath` trailer overstated what is no longer read. Fixed as proposed (`eedb8989b`).
- fix-proposal, LOW: the `TestPushPairAnchored_PushesBothSidesRetriesAndReportsEachSide` trailer left out a removed negative check. Fixed as proposed (`eedb8989b`).
- fix-affected-set: no findings.

## Disputed

## Deferred

## Rulings
