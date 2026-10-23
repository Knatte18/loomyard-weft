# Card 21 — Webster's plan-conflict recurrence names the amendment route

**Edit:**
- `internal/shedadapters#planConflictWayForward`
- `internal/shedadapters#TestWebsterProducer_OutcomeStuck`
- `internal/shedadapters#TestWebsterProducer_ConflictRerouteCapThenRepeatHalts`
- `internal/loomcli#loomRerouteCapWayForward`
- `internal/loomcli#TestLoomRerouteCapWayForward_IsRowNeutral`
- `internal/websterengine/doc.go`

**Intent:**
D7: Webster's plan-conflict recurrence halt names the amendment route as its way forward instead of a hand edit.

`planConflictWayForward` becomes: the run's parent records the fix as a decision naming the conflicting pending cards by slug and number with `lyx loom decision add <slug> --by parent … --amends-plan`, then runs `lyx loom goto --to Plan-Burler-Review` and `lyx loom resume` in the task worktree.
Both reasons that end with it, the first reroute's "if plan review does not clear it" and the recurrence halt, keep their lead text.
The route's steps succeed from the halted state: the plan is approved, so `--amends-plan` is admitted, and the record makes the settled guard admit the goto (cards 13 and 14).
`internal/websterengine/doc.go`'s plan-conflict paragraph says the recurrence halts for that route rather than a hand edit of the named pending cards.

`loomRerouteCapWayForward`, the clause a reroute-cap halt ends with, names a hand edit of the pending cards for a Webster plan conflict too; its Webster clause becomes the same amendment route, and its parent-branch clause stays.

`TestWebsterProducer_OutcomeStuck`'s recurrence row asserts the reason names `--amends-plan` and `lyx loom goto --to Plan-Burler-Review`; its existing `lyx loom resume` assertion stays.
`TestLoomRerouteCapWayForward_IsRowNeutral` asserts the clause names the amendment route in place of the hand edit, and `TestWebsterProducer_ConflictRerouteCapThenRepeatHalts`'s `capWayForward` constant, which copies the clause, takes the new text.

**ImpactSummary:** two constants' text: the Webster producer's two Stuck reasons and the reroute-cap halt clause; the three tests that pin them are updated here.
