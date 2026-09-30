# Webster-Review round 1 — fixer report

All seven findings are fixed, one warp commit each, none pushed.
After the last fix, `go build ./...`, `go vet ./...` and `go test ./...` pass.
`go test -tags integration` over fabricengine, websterengine and landingshed also passes.

## Fixed

- **F1 (MEDIUM), `48b60f53a`**: rewrapped every new or changed Go comment the diff added to semantic line breaks.
  The comments are in loomshed, landingshed, fabricengine, loomcli, loomengine, loomrecipe, planparser, shedrecipe and websterengine, including the test files.
  Untouched legacy lines were left alone.
  While editing `internal/landingshed/doc.go`, I also corrected its package comment, which still named "two general producers, Publish and Finalize" after PR-Gate became the third.
  That line was not in the diff, but the diff made it stale.
- **F2 (LOW), `4d8702568`**: `PendingRejection`'s doc comment now names `battenshed.ChildDecision`.
- **F3 (LOW), `6ac06e109`**: `PRGate` gained a `done(ctx)` helper that consults `cancelErr` first, and all three Done exits use it.
  New test: `TestPRGate_CancelledBeforeDone_IsError` cancels the context inside `TaskHead` and asserts that a matching approval's exit reports `context.Canceled`.
- **F4 (LOW), `a65a6750a`**: removed the rework stencil's "or an explicit entry in the coverage file" escape.
  It now states that a round with no new card is rejected, so a coverage entry alone never answers a finding.
- **F5 (NIT), `273b9b66b`**: `finish`'s doc comment now says it runs step 6, and that the absent-record crash path calls it with `clear` false.
- **F6 (NIT), `12a0d8486`**: `ReadDecision`'s comment now says both records coexist only when `approve` and `reject` race, and that the approval then wins, as it does in the gate when both match.
- **F7 (NIT), `134fd7b28`**: the batten paragraph in `docs/overview.md` names `lyx loom approve` or `lyx loom reject`.

## Deferred

None.
