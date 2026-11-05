# Fixer-report, round 1 (code)

Every finding is fixed.
`lym gate test` passes over `boardengine`, `loomshed`, `loomcli`, `shedrecipe`, `battencli` and `envkit` (tier 0).

## Changes

- **F1, fixed as proposed, with one narrowing from the self-check.**
  - `internal/boardengine/raddlefile.go`: `FileRaddle` skips `f.Check` and the entry writes when the board already holds every filed slug as a discussed entry whose body begins with the part or project body prefix, and then only writes the store, `store.json` last.
    The prefixes are exported as `RaddlePartBodyPrefix` and `RaddleProjectBodyPrefix`, which `loomshed` also builds the bodies from, so the writer and the match share one source.
    `Board.HoldsRaddleFiling` answers the same question for the seam.
  - `internal/loomshed/raddlefile.go`: the `RaddleFileDeps.AlreadyFiled` seam, asked only when the store is not done and the proposal parses.
    When it is true, `verify` is skipped and `stageAndFile` still runs, so the filing and the store are written.
  - The seam is wired in `internal/loomcli/raddlewiring.go` over the hub board, required in `internal/shedrecipe/entries_raddle.go`, and filled in the `shedrecipe` fixture, `envkit` and the `battencli` exemption map.
  - Tests: `TestFileRaddle` rows `ARetryAfterALostStoreWriteWritesTheStoreWithoutTheCheck` and `ADifferentHashOrCreatedStampIsNotTheDoneMarker`, `TestHoldsRaddleFiling`, loomshed rows `AHalfDoneFilingIsFinishedWithoutTheChecks` and `AHalfDoneFilingOfAnEditedProposalIsCheckedAgain`, and a `FileNilAlreadyFiled` row.
  - The spec `contracts/specs/raddle-proposal-spec.md` names the retry exception.
- **F2, fixed as proposed.** `docs/overview.md`: the `_lym/raddle/` row describes a raddle run's directory, a `_lym/usage/` row is added, and the sentence about the raddle directory no longer points at the board's note.
- **F3, fixed as proposed, with the premise partly false.**
  The table loop of `TestFileRaddle` already asserted that a second call with the same hash writes nothing and runs no second check, so the marker branch had a row.
  The missing parts are added: the different hash and the different created stamp rows, and `TestRaddleFileDepsStoreDone` over a temp board directory with a `store.json`.
  Its store record lists the run's slug in `Filed`, so a row for another Sketch fails without the closure's `held.Sketch == slug` clause.
- **F4, fixed as proposed.** `raddleApproveVerb` refuses when `NoticeSession < Session`, naming `lym loom raddle-propose` as the way forward; the approve table seeds `NoticeSession` and has a new row for the missing notice.
- **F5, fixed as proposed.** `recordWrite.Call` reads the stored record whenever the record or the log is missing, and picks the stored-record support log whenever one is found; a new row covers a present record with a missing log.

## Self-check

Lens reports are saved beside this file as `round-1-fixer-report.lens-<lens>.md`; no fork returned without a report.

- fix-code-correctness, LOW: no row pinned the closure's `held.Sketch == slug` clause. Fixed: the store record's `Filed` now holds the run's slug.
- fix-consistency, MEDIUM: the pinned spec says Raddle-File runs the re-check and files only an approved hash, with no retry exception. Fixed: the spec names the exception.
- fix-consistency, LOW: the `run.json` and `landed.json` names in the `_lym/usage/` row are unconfirmed. Disputed, `premise false`: `internal/shedrun/usage.go:20-21` declares `run.json` and `landed.json`.
- fix-proposal, MEDIUM: a retry skipped the approval match for a proposal edited after the crash, and the store would record the edit. Fixed with my own change: the resume applies only when the stage directory still holds the proposal's hash, so an edited proposal runs every check again.
  The reviewer's proposal alone is not made, since it trusted the slugs and the body prefix only.
- fix-proposal, LOW: the resume dropped the landed entries from the parent notice. Fixed: the resume calls the re-check, ignores its failures and keeps its landed list.
- fix-proposal, LOW: the `RecordWithoutSupportLogGetsTheLog` failure message said the board is not read. Fixed: it says the entry's body is not read.
- fix-affected-set: no findings.

## Disputed

- The fix-consistency LOW on the `run.json` and `landed.json` names, ground `premise false`, evidence `internal/shedrun/usage.go:20-21`.

## Deferred

Nothing deferred.
