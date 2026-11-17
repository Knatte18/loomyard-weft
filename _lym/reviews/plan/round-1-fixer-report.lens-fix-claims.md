```yaml
lens: fix-claims
outcome: reported
```

Lens: fix-claims. I checked every sentence the fix added or rewrote that states a fact about the code. One finding.

**MEDIUM: the overview says a zero `StalemateRounds` "runs no stalemate check", but Card 10 says it does**
- Changed text: `_lym/plan/00-overview.md`, the Decision line of nil-seams-keep-today: "a nil `BouncerConfig.Snapshot` or a zero `StalemateRounds` runs no stalemate check". This wording came from F8's proposed fix.
- Problem: the claim contradicts Card 10 in two places.
  - Card 10's `stalemated` bullets check first for a round whose own escalation record holds `stalemate`. That check runs "whatever `StalemateRounds` now is", and only the next bullet returns false when `StalemateRounds` is 0.
  - A Card 10 test row expects the replay to escalate `stalemate` again "with `StalemateRounds` set to 0".
- A nil `Snapshot` doesn't skip the check either. It still runs and counts every round as changed, so it never escalates.
- Suggested fix: "a nil `BouncerConfig.Snapshot` or a zero `StalemateRounds` never starts a stalemate escalation".

These claims hold:
- **Card 1:** `newIntegrateBuild` seeds its index with `read-tree` (`integrateweft.go:126`). Without that seed, `git add -A` would drop a tracked file that an ignore rule matches.
- **Card 2:** `checkReviewKeys` refuses stray top-level keys (`verdict.go:184-200`). The stray-key way forward matches line 42's format.
- **Card 6:**
  - `priorRoundsBlock` holds the first-round and clean-room strings (`prompt.go:366-390`).
  - The stencil has no conditionals.
- **Card 8:** the help text's "only top-level keys" sentence is at `validate.go:32`. The `validateReviewVerb` comment and the file header say what the finding quotes.
- **Card 10:**
  - escalation.go's header and `ErrEscalationMalformed` both say four causes.
  - The doc.go sentences match the quoted text.
  - The fail-loud-guard wording matches `softCapRule`'s own comment.
- **Card 11:**
  - A recorded accept reaches `acceptRound` (`bouncer.go:1273`), which calls `writeCarryOver`.
  - The arbiter's default branch refuses `budget`.
- **Card 12:** `docs/overview.md:1279` names "a circling design-wrong finding or at the hard cap".

Tool call I made:
- `bouncer.go:1255-1275`: confirms that a recorded `accept` on an escalation goes through `acceptRound`. My context only held `acceptRound`'s body, not the code that calls it.
