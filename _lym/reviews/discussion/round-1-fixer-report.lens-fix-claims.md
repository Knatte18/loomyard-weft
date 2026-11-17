```yaml
lens: fix-claims
outcome: reported
```

Lens: fix-claims. I checked each claim the fix added or rewrote against the code at HEAD.

Claims that hold:
- `opts.DecisionsRouted` is set exactly when the parent is notified or decisions are escalated (`internal/shedadapters/burlerfix.go:75`).
- `acceptReview` and `acceptedReview` both run `ParseCurrentReview`.
- A `PassOnCap` entry is let through at its cap (`internal/shuttleengine/gate.go:76-82,112-113`).
- `arbiterstop` admits hard-cap, circling and decision today, so adding `stalemate` is a stated change.
- The auto-accept opens `settleUndecided`.
- The `Amendment` mark is read per run directory for the whole generation (`amendment.go:82-89`).
- `### Added after Discussion (<by>, <date>): <title>` is a real heading form (`internal/discussionparser/identity.go:119`).
- The parent ask is a per-round option that a gate entry receives (`parentseat.go:51-54`), so a round-check entry can be given it.

Findings:

- MEDIUM. Changed text: D2, "every `Added after Discussion` entry `lym loom decision add` recorded since round N-1's review".
  Summary: no on-disk value can tell which entries came after round N-1's review.
  Evidence: the heading carries only `(<by>, <date>)` (`internal/discussionparser/identity.go:119`), so a date cannot order an entry against a review written the same day.
  Unless the record names a stamp the entry carries or a count taken at review time, "since round N-1's review" has nothing to be computed from.

- LOW. Changed text: D6 acceptance criterion, "an unchanged round at the soft cap with `AutoAccept` set is auto-accepted, not escalated with cause `stalemate`".
  Summary: the criterion leaves out the auto-accept's other conditions.
  Evidence: `internal/shedadapters/bouncer.go:1355` requires `spawned && readErr == nil && !escalated && !b.blockingSendsToRule(round)` besides `softCapReached && AutoAccept`.
  When one of those fails, the round falls through to the stalemate check, so the criterion as written does not always hold.

Tool calls:
- Read the `PassOnCap` lines in `internal/shuttleengine/gate.go`: my context lacked the cap semantics the D1 claim relies on.
- Read `gate.go:74-82` and grepped for the `Added after Discussion` heading, the decision-cause sites and the parent-ask sites: my context lacked the decision-entry heading format and how a parent ask reaches a round.
- Grepped for `let_through` in `internal/shuttleengine/gate*.go`: to confirm that a capped entry counts as let through.
