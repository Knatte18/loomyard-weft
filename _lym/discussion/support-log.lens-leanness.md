```yaml
lens: leanness
outcome: reported
```

Task: a leanness review of the decision record and support log, looking for content to move or cut.

**Findings**

1. MEDIUM — D2, the "Rationale" paragraph's second sentence. "A step before Plan with its own round would add a review-fix loop the perch already provides" argues against a rejected alternative, and the spec says `## Decisions` carries Decision + Rationale only. The same point is already the first bullet of the support log's `## Rejected alternatives`. Fix: cut the sentence from D2.
2. MEDIUM — D1, the "Rationale" line. "A second mechanism for the same seat would duplicate ... assist.go" argues against the rejected "new agent-fan key" alternative, which the support log already lists. Fix: cut it. D1's body already says no key or code is added and why.
3. LOW — D6, the "Rationale" line. "A fork that has to report back to the orch would put it back there" re-argues the rejected alternative that the support log carries. Fix: keep only "the operator asked that a busy orch can keep the review off its main thread".
4. LOW — D1. The sentence describing how the assistant's findings are judged (same skepticism, `origin: assist:assist`) restates existing behaviour from `burler-assist-consolidation.md`. The plan writer needs only "the existing assistant is the agent fan; no code; D6 names the keys". Moving the description to the support log is optional, since it carries D1's premise. By the completeness-before-leanness test, keep one clause that findings join the round as lens findings do.
5. LOW — `## Constraints`, the last bullet ("Model-spec `fable[high]` is a valid assistant model…"). It is a researched fact D1 rests on, so it belongs in the record and should stay. It would sit better inside D1, next to its claim, than among the PATTERN constraints. This is placement only.
6. LOW — `## Auto-mode assumptions`. All three bullets restate picks already stated in D2, D5 and D7: the 30-minute deadline, later rounds asking only after findings, and `BLOCKING` as the default severity. That repeats the Decisions, against "say it once". Fix: cut them, or reduce each to a pointer, since the ledger already marks them as auto-picks.
7. LOW — `## Notes for the plan writer`. The second bullet, on PR-Review's notifier wiring, duplicates the `internal/loomcli/landingdeps.go` entry in `relevant.md` and the D3 mention of `orchcli.NotifyParent`. It can be cut.
8. LOW — the support log, `## Interview`, Turn 2. It restates the Decisions one-to-one with no operator content. That is acceptable for an autonomous log, but it adds nothing beyond the ledger.

Other things noticed, not leanness:

- D3 calls the ask record "named only through `lymdirs`". That is plan-level detail already covered by the `PATTERN-lym-files` constraint, so it is redundant in D3.
- The D1 acceptance bullet says "existing coverage may meet it". That is fine per the spec, but it is weakly testable as a criterion for a no-code decision.

No further tool calls were made.
