```yaml
lens: fix-way-forward
outcome: reported
```

Checked the ways forward this fix introduced: the D6 halt's `lym batten cancel`, D2's `circling continue` and `resume` paths, and the new criteria.

**[MEDIUM] `lym loom circling continue` cannot be run after a hard-cap escalation**
- **Changed text:** the new D2 bullet "An escalation raised again is answered as any other: `lym loom circling continue` runs the next round over the new proposal, and `lym loom circling accept` files the new proposal…". The new criterion "D2: after a goto to Raddle-Write from a perch whose last round escalated, the perch raises that escalation again, and `lym loom circling continue` then runs the next round over the new proposal" depends on it.
- **Summary:** if the recorded round escalated at or past the hard cap, `circling continue` runs no round. The only answer left is `circling accept`, which files the new proposal without any round reviewing it. Neither the bullet nor the criterion names this case.
- **Evidence:**
  - `internal/shedadapters/bouncer.go:1257-1261`: `settleUnconverged` handles a `CirclingContinue` at the hard cap with `retireContinue` and then `b.escalate(round, EscalationHardCap, ptr)`. It escalates again and returns no Stuck that would lead to a next round.
  - `bouncer.go:1603-1604`: the escalation text at the hard cap says "it can only be accepted" and offers only `lym loom circling accept`.
  - So after a goto from a perch that escalated at the hard cap, the criterion as written cannot be reached. The bound should say that in this case the only way forward files the new proposal unreviewed. The other ways are to restrict the criterion to escalations below the hard cap, or to name another way to get a review.

The D6 halt's way forward holds. `lym batten cancel &lt;slug&gt;` runs from the prime or a beam whatever the run's state (`internal/battencli/cancelcmd.go:47-60`). It clears a run-form status (`cancel.go:91`) and refuses only a beam project with work left under it, which the converted project is not. So the halt leads to a state that can make progress. `lym loom resume` after a budget halt also holds: `run.go:648` says "A resume writes no entry, so it grants one more budget". I also checked whether the D6 halt's "entry already of kind `project`" test would fire on a `raddle` run's Sketch entry. It would not: `sketchOf` sets `KindNote` (`internal/boardengine/sketch.go:56`).

Tool calls:
- grep of `internal/battencli` for cancel functions: my context did not show where batten cancel is implemented.
- `sed`/grep of `cancelcmd.go` and `cancel.go`: my context lacked batten cancel's preconditions and board effects.
- grep of `sketchOf` in `internal/boardengine`: my context did not show what kind a Sketch entry has.
