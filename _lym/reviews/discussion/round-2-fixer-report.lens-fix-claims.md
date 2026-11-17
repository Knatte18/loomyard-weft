```yaml
lens: fix-claims
outcome: reported
```

Task: check, through the fix-claims lens, every claim the fix added or rewrote against the code at HEAD.

**LOW** — D6, "every later entry for round N, a replay included, reads the recorded value"
- **Summary:** a replay of a round that stands as recorded never enters `settleUnlessRefused`, so "entry" does not cover that replay.
- **Evidence:** `internal/shedadapters/bouncer.go:504-506` reads `if b.standsAsRecorded(n) { return b.settle(ctx, n, false) }`. That path goes straight to `settle`, and through it to `settleUndecided`, where D6's check sits. The value has still been recorded by then, because a round's first judgment always passes through `settleUnlessRefused` (`bouncer.go:433`, `457`, `508`, `1985`). The behaviour holds; the sentence names the wrong entry point. "Every later settle of round N reads the recorded value" would be accurate.

Checked and holding at HEAD:
- **"A settled decision file is no source: one is settled only on an accept, which ends the segment."** Holds: `circling.go` refuses a settled non-accept, and `acceptRound` returns `shedengine.Done` (`bouncer.go:1313-1330`).
- **"It records titles, as `writeDecisionSnapshot` does."** Holds: `bouncerdecision.go:291-314` stores `decisionSnapshot{Titles: titles}`.
- **"A `lym loom decision replace` fold removes added entries."** Holds: `contracts/specs/loom-discussion-spec.md:132` lets replace remove an Added after Discussion entry, and the parent directive has a replace fold every added entry.
- **"Go cannot tell a finding the parent raised from one tagged with the parent's id."** Holds: `verdict.go:73-85` says both `Origin` and `ParentIDs` are never validated.
- **The D6 claim that recording at the first entry to `settleUnlessRefused`, ahead of `routeDecisions`, covers every judged round.** Holds: every first settle of a judged round goes through `settleUnlessRefused` (`bouncer.go:433`, `457`, `508`, `1985`), and `routeDecisions` is its first statement (`bouncer.go:1038`).
- **"The decision escalation keeps its place after the judge and ahead of the refusal check and the settle step."** Holds: see `bouncer.go:1037-1044`.

Further tool calls:
- A grep of the specs for the `decision replace` fold wording and of `internal/shedadapters` for the `settleUnlessRefused` and `settle` call sites. My context had neither the spec line nor the call-site list.
- A read of `bouncer.go:495-510` and `715-750`, the direct `settle` replay path and `resettle`, to see whether any first settle bypasses `settleUnlessRefused`. My context lacked those spans.
