```yaml
lens: code-tests
outcome: reported
```

Lens code-tests: for each behaviour the diff adds or changes, the test row that would fail if it broke.

| Behaviour | Pinning row | Verdict |
|---|---|---|
| An answered ring with no turn end read after it gets no bell inside the 30 s window. This is the new `woken` hold. | `MailAfterAnAcknowledgedRingGetsNoBellBeforeTheWindowWhileTheWokenTurnHasNotEnded` (`deliver_test.go:2406`). | Pinned. Before the diff the bell came at about 5 s, so the `< 30s` assertion fails without the hold. The row reaches a stale `TurnEndedAt` (ring time minus one minute) through the `creelfake` reading path, as `Idle` does in production. |
| The window still ends the hold, so the bell comes at or after 30 s on the `woken` path. | The same row. | Pinned, assuming `untilBell` fails when no bell arrives. I did not read `untilBell` at `deliver_test.go:1837`, so that is unchecked. |
| A turn end read after the ring clears `woken`, so the 5 s grace applies as before. | `MailAfterAnAcknowledgedRingGetsTheBellAfterTheGraceOnceATurnEndFollowsTheRing` (`deliver_test.go:2415`). | Pinned. It fails if `woken` stays true after a later turn end (bell after 30 s) or if the grace is dropped (bell before 5 s). |
| An unanswered ring is held for the window, then belled at once. This path was reordered. | `ARingUnansweredForTheWindowIsFollowedByTheBell` (`deliver_test.go:2349`). | Pinned. |
| A ring answered by a claim or a running turn gets no bell. | `ARingAnsweredByAClaimIsNotFollowedByTheBell` (`:2360`) and `ARingAnsweredByARunningTurnIsNotFollowedByTheBell` (`:2385`). | Pinned. Both keep a waiter registered, so they reach the grace branch and not the new hold. |
| The grace with no waiter is unchanged. | `AnIdleStrandWithNoWaiterGetsTheBellOnceAfterTheGraceAndAgainForANewPost` (`:2288`) and `AWaiterThatEndsWithoutARingStartsTheGraceAgainFromItsEnd` (`:2425`). | Pinned. `mark.at` is zero in these, so `woken` is false. |
| A zero `TurnEndedAt` after a ring holds the bell for the window. Either the pane decided, or a turn is running, or the operator interrupted. This is the Bound line added to `doc.go`. | None. | Gap, finding below. |

**Finding 1 — LOW, class `design`**
- **Location:** `internal/creelengine/deliver.go:287`, with the new Bound line in `doc.go`.
- **Summary:** no row pins the zero-`TurnEndedAt` branch of `woken`. Both new rows set a non-zero `TurnEndedAt` (`ringAt.Add(-time.Minute)` and `ringAt.Add(time.Second)`). That means the case the diff documents as a bound is untested: after an answered ring, with `SetTurn(alpha, false, time.Time{})` or a running turn, mail that arrives later gets no bell until the window ends. An implementation using `idleness.TurnEndedAt.Before(mark.at)`, which treats zero differently, would pass every row.
- **Evidence:** the `SetTurn` calls at `deliver_test.go:2408` and `:2417`. The running-turn row at `:2385` registers a second waiter, so it reaches the grace branch and never the `woken` branch.
- **Suggested fix:** one added row, or an extra assertion in the first new row, with a zero `TurnEndedAt` after an acknowledged ring and a new post, expecting no bell before 30 s.

No finding on test economy. The two new rows cover distinct branches of `woken`, and no existing row covers either one.

Tool calls: one `Read` of `deliver_test.go` lines 2180–2429, because the manifest held only the diff hunks of the new rows. One `grep` for the `ackedRing` and `untilBell` definitions, which gave their locations (`deliver_test.go:1795` and `:1837`) but not their bodies.
