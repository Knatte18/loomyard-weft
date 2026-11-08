# Fixer-report, code review round 1

## Changes

All three findings were fixed as proposed, in commit `8ec425cf` plus a comment follow-up commit.
The three share files (`cli.go`, `describe_test.go`, `land_test.go`), so they went in one commit instead of one each.

- F1: `DraftBeamDescription` stats the description path first and refuses when a file stands there.
  The refusal names the file, `lym beam describe <slug>` and deleting the file for a fresh draft.
  `BeamDraftDeps` gained a required `Slug` field so the refusal can name the verb; `beamDraftDeps` in `deps.go` fills it.
  The `AnExistingDescriptionIsReplacedWhole` row became `AnExistingDescriptionIsRefusedAndLeftIntact`, which also asserts the existing bytes are unchanged.
  The `--draft` help and flag text, the `describe.go` comment, `beamcli/doc.go` and the doc comment in `beamdescribe.go` match.
- F2: `landWorld` gained a `landingErr` field that fails the `Landing` seam.
  A new `TestDescribe` row records successfully, then fails the Landing read, and asserts "is recorded, but the landing record is unreadable" and "lym beam pr update b".
- F3: `describeCmd` now builds on `describeCmdOver`, which takes the function that runs the verb.
  A new test, `TestDescribeCmd_DraftFlagReachesTheDraftSeam`, runs the cobra verb with and without `--draft` over the fake world.
  It asserts the draft seam ran and the record seam did not, and the reverse.

Self-check follow-up: the `DraftDescription` seam comment in `internal/beamcli/land.go` still said "over", so it was reworded.

Gate repair: the gate refused two comment lines of this round that reached 120 characters (`describe_test.go` and `beamdescribe.go`).
Both were split at sentence or clause boundaries; the text is unchanged.

Gate: `lym gate test ./internal/landingshed ./internal/beamcli` passed (tier0, 978 passed, 0 failed) before the follow-up.
The follow-up is a comment change only.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency, LOW, `land.go:36` comment still says "over": fixed as proposed.
- fix-consistency, LOW, `role-beams.md` unchecked: read it; line 33 says `--draft` "first writes a starting point", which agrees with the refusal. Nothing to change.
- fix-proposal, LOW, a truncated draft left by a failed write refuses a re-run: fixed differently by leaving it.
  The refusal's way forward already names deleting the file, and recording validates the text, so a truncated draft is not recorded.
- fix-proposal, LOW, `describeCmd`'s production closure is untested: disputed, `every fix worse`.
  The closure is one forwarding call, and testing it needs the real hub deps that `onHubLandDeps` builds.
- fix-affected-set, LOW, `land.go:36` stale comment: duplicate of the fix-consistency finding, fixed.
- fix-affected-set, LOW, `role-beams.md` unchecked: duplicate of the fix-consistency finding, nothing to change.

## Disputed

- fix-proposal LOW on the untested production closure of `describeCmd`. Ground: every fix worse.
  Evidence: `describeCmd` is `describeCmdOver(func(...) { return onHubLandDeps(cmd, "lym beam describe", ...describe(cmd.Context(), deps, slug, draft)) })`.
  A test of it needs a real hub for `onHubLandDeps`, and the integration tier already covers the hub's land deps.

## Deferred

None.

## Rulings

None.
