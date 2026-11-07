# Fixer-report, round 1

## Changes

Both changes are in `internal/hubreconcile/ensure_integration_test.go`, in `TestEnsure_HubWideWalkRendersAndCommitsTheHeddleOverview`.
The table gained the fields `failedWriteFirst`, `renderFails`, `wantOverviewCommitted` and `wantLandingCommitted`, and its `seed` takes the hub.

- **F1, fixed as proposed.**
  Row "heddle store that does not load" commits a `heddle.json` that fails to decode, and a retired landing key.
  It expects `Ensure` to return nil, `HEDDLE.md` to stay as it was, the landing config to be committed, and `RenderOverview` to still fail.
  The failed-board-write steps run for the first two rows only.
- **F2, fixed as proposed.**
  Row "overview already current" commits the overview as `RenderOverview` writes it, plus a retired landing key.
  It expects the walk's commit to name the landing config and not `HEDDLE.md`, and `RenderOverview` to report no change afterwards.

Commits: `73c14705` (F1, F2) and a follow-up that reuses `seedRetiredHubWideKey` for the landing seed (self-check, consistency lens).
The tier-1 gate for `./internal/hubreconcile` passed after each.

## Self-check

- fix-code-correctness, LOW: row "overview already current" passes against an always-append mutant. Disputed, `every fix worse`: see below.
- fix-proposal, MEDIUM: the same observation as the line above, raised against F2's proposal. Disputed, `every fix worse`: see below.
- fix-consistency, LOW: `commitRetiredLandingKey` duplicated `seedRetiredHubWideKey`. Fixed: `seed` takes the hub and the rows call `seedRetiredHubWideKey`.
- fix-affected-set: no findings.

## Disputed

- **F2 residue (both lens findings above), ground `every fix worse`.**
  The always-append mutant is observably identical to the code: the staging layer drops an unchanged path, so the commit, the file and `RenderOverview` agree either way.
  The review says as much: "The staging layer drops an unchanged path anyway, so an always-append mutant would pass."
  The Goal's observable half, that an identical render commits nothing, is pinned by the row.
  Telling the two apart would take a production seam or a log assertion in `renderHeddleOverview` for no behavior a caller can see, which makes the code worse.

## Deferred

None.
