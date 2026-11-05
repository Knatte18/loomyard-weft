```yaml
lens: fix-claims
outcome: reported
```

fix-claims: checked every sentence the fix added or rewrote against the code at HEAD.

no findings

What I checked, each claim holding:
- **Card 10, the dobby-stop marker:**
  - `landChecks` is the only writer of the marker (`internal/pawlengine/beamawait.go:317`; no other production write).
  - `accept` removes it only after a pass (`beamawait.go:306`), `refuse` only after a successful restart (`beamawait.go:348-354`), and the lifted-hold branch removes it (`beamawait.go:191`).
  - A `refuse` whose restart failed leaves it standing (`beamawait.go:347-356`).
  - A resume rewrites the board status before the row runs (`internal/shedengine/run.go:317-318`).
  - In the failed-restart case the ready record still stands, since the dobby lands only a flagged beam and ready is final. So "the entry then re-runs the re-check" holds.
- **Card 15, refusal 5:** the only `Halt` that `ResolvePullRequest` returns comes from a missing origin fact (`internal/landingshed/pullrequestmode.go:100`, reached only when `checkOrigin` finds one). The rules-read halt lives in `resolveWithParentRule`, which ready does not call.
- **Card 15, refusal 6:** `PullRequestSide` builds the API through `pullrequest.Open` over the origin facts (card 13 line 42). So no usable token or a changed origin surfaces as its error.
- **Card 3:** for a required check, GitHub counts `success`, `neutral` and `skipped` as satisfied, and `stale` is a valid check-run conclusion. The retired-row move calls `pullrequest.Open` (card 8 line 35), so the new caller sentence for `pullrequest/doc.go` holds.
- **Card 12:** `awaiting` is a real run state (`internal/shedengine/status.go`, `activity.go:40`). A Done outcome keeps the row's own name with state done only for an empty `on_done` (`run.go:470-492`).

Calls made:
1. One grep over the run states in `internal/shedengine`, the `pullrequest` use in card 8 and `PullRequestSide` in cards 13–14. The context had not confirmed that `StateAwaiting` exists, that card 8 calls `pullrequest`, or what `PullRequestSide` opens.
