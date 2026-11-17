# Card 9 — The parent notice names each cause's way forward

**Create:**
- `internal/shedadapters#Bouncer.noticeWayForward`

**Edit:**
- `internal/shedadapters#Bouncer.renderEscalation`
- `internal/shedadapters#TestBouncer_DecisionEscalation`
- `internal/shedadapters#TestBouncer_Circling_NoDecisionAwaitsNamingBothVerbs`

**Uses:**
- `internal/shedadapters#decisionBouncer`
- `internal/shedadapters#layoutCirclingBouncer`

**Intent:**
D2 and D6: the one-line parent notice names how to settle the escalation, not only the brief.
Today the notice is cause-blind: `bouncer-template-parent-notice.md` filled with the slug and brief path.

`renderEscalation` appends `noticeWayForward(round, cause)` to the filled notice, after one space, so the notice stays one line and the stencil keeps its two markers.
`noticeWayForward` returns:
- for cause `decision`: `Way forward: record each answer with "lym loom decision add <slug> --by parent", one entry per answered finding, then decide with "lym loom circling accept <slug>" or "lym loom circling continue <slug>".`;
- for every other cause: `Way forward: decide with "lym loom circling accept <slug>" or "lym loom circling continue <slug>".`;
- at or past the hard cap, either text with the continue clause dropped, as `escalationReason` does, since `circlingVerb` refuses `continue` there.
An empty slug already fails the notice stencil's fill, so such a round escalates with no notice as today; the clause is appended only to a filled notice.
A message, or a `lym loom decision replace` fold, adds no entry, so a held finding answered that way stays open (D2).

Tests, tier0:
- `TestBouncer_DecisionEscalation` asserts the escalated round's `ParentNotice` names `lym loom decision add` and "one entry per answered finding".
- `TestBouncer_Circling_NoDecisionAwaitsNamingBothVerbs` asserts, on its rows with a slug, that the `ParentNotice` names `lym loom circling accept` and `lym loom circling continue`, and gains a row at the hard cap (`HardCap` set to the escalated round through the `circlingBouncer` tweak) whose `ParentNotice` names `lym loom circling accept` and not `continue`.
Both assert the Go-filled clause, not stencil prose.

**ImpactSummary:** Every escalation's parent notice gains one Go-written clause; the record, brief and Awaiting reason are unchanged.
