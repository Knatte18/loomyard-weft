# Fixer-report, code review round 1

## Changes

- **F1** (MEDIUM), fixed differently.
  - Proposed: skip a member whose slug has a listed past-Land run or landing-notice memory.
  - Made: `flagMembers` flags an open member only while a run of its slug is listed ahead of Land (`internal/dobbyengine/landnotice.go`).
  - Why: the proposed skip leaves the double line when the run drops from the listing while the entry is still open, and the memory is already gone by then.
    A run is listed ahead of Land only until it lands, so the flag is never set again afterwards.
  - Row `AMemberWhoseEntryStaysOpenPastLandIsNotFlaggedAgain` keeps the entry open past Land and ends with the run unlisted; it expects one line and one flag call.
- **F2** (MEDIUM), fixed as proposed.
  - `TestHubDobbyDeps_PullRequestRecordedReadsTheBeamLandingRecord` in `internal/beamcli/dobbydeps_integration_test.go` calls the seam over no record, a record without a pull request and a record with one, expecting false, false and true with a nil error.
  - I added it beside the existing seam test, not as rows in it.
- **F3** (LOW), fixed as proposed, with a recorded membership.
  - `LandNoticeState.Member` is set while the run is ahead of Land.
  - A remembered member's landed line goes out under per-task whenever a follow-up is set, flagged or not.
  - `ClearOwed` stays tied to the entry carrying the field.
  - Row `AMemberLandingInTheTickThePullRequestIsRecordedGetsTheLine`.
- **F4** (LOW), fixed as proposed.
  - New `TestLandNotices` rows: non-member, done member, failed flag write retried, landed-without-commit line with the follow-up, done entry with the follow-up, and `land_notice: off` with a pull request.
  - New harness fields: `pullRequestFromTick`, `entryProject`, `entryDone`, `entryStaysOpen`, `failFlags`.
- Doc upkeep from the self-check: the file header of `landnotice.go` and the landing-notice section of `internal/dobbyengine/doc.go` now state the flagging condition, `Member` and the unflagged-member line.

Gates: `lym gate test ./internal/dobbyengine` (tier 0) and `lym gate test ./internal/beamcli --tier 1` passed.
The tier 1 run preceded the last `dobbyengine` edit, which changed nothing in `beamcli`.

Commits:

- `beamcli: fix F2`
- `dobbyengine: fix F1, F3, F4`
- `dobbyengine: fix F1 — flag a member only while its run is listed ahead of Land, and bring the docs level`

F1, F3 and F4 share one file pair and were committed together.

## Gate repair

The gate found comment lines of `landnotice.go` and `doc.go` that reached 120 characters or wrapped mid-sentence.
Each was broken at sentence boundaries, one sentence per line, with no change to the wording's meaning.
Commit: `dobbyengine: fix comment line breaks the gate found`.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency, LOW (file header stale): fixed with my own change, the header now says members ask for it flagged or not.
- fix-proposal, MEDIUM (a run dropped from the listing re-opens the double line): fixed with my own change, see F1.
- fix-proposal, LOW (a never-flagged or done member gets a line): disputed, every fix worse.
- fix-proposal, LOW (a clear owed from a stale `Requested`): disputed, premise holds but it matches the earlier behavior and the clear is idempotent.
- fix-affected-set, LOW (`doc.go` stale): fixed as proposed.

## Disputed

- fix-proposal LOW, a never-flagged or done member gets a line.
  - Ground: every fix worse.
  - Evidence: F3 asks for a line to a member that was never flagged.
    Narrowing it to open members would drop the case F3 names, where the entry is already done and unflagged when the pull request is first read.
    A done entry is already read as landed whatever set its status (`takeDoneEntries`).
- fix-proposal LOW, a clear owed for a stale `Requested`.
  - Ground: every fix worse.
  - Evidence: `ClearNotifyOnLand` on an entry without the field is a no-op write, and `note.Requested` is not refreshed once the entry is off the board.
    Dropping the owed clear would leave a pruned entry's flag on the board, which `rememberRuns` guards against by design.

## Deferred

None.

## Rulings

None.
