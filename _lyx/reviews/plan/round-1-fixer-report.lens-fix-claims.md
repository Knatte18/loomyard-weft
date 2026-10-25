```yaml
lens: fix-claims
outcome: reported
```

No findings.

Tool calls:
- `internal/shedadapters/bouncer.go:480-510` and grep for budget-cause writers: to check card 12's settled-accept replay claim (F4) and card 13's "only an older binary or a flipped switch leaves" claim (F6). My context didn't have the replay branch or `recordAutoAccept`/`recordAutoContinue`. Both write from inside the Bouncer, after the routing.
- `internal/shedadapters/circling.go:80-96`: the doc for those two writers, for the same F6 claim.
- grep of `contracts/stencils/bouncer/bouncer-template-escalation.md`: to confirm card 13's claim that the other cause's brief points the parent at the round's review. Line 23 names `{{.review_path}}`.
- grep of `internal/shedengine/doc.go` for budget: to check card 12's claim that `lyx loom resume` re-judges after a dropped-key budget halt (F5). Line 50 says a resume of a budget halt grants one more budget.
- grep of `internal/shedadapters/bouncer.go` for `rejudge` (twice): same F5 claim. Its doc says it archives the judge outputs so the next Call re-judges the round.
