# Card 3 — keepalive-pings

**Create:**
- `plan:internal/parentreview#maxKeepAlives` -> `const maxKeepAlives = 3`
- `plan:internal/parentreview#Store.RecordPing` -> `func (s Store) RecordPing() error`
- `plan:internal/parentreview#closures.pingDue` -> `func (c *closures) pingDue(d Delivery, now time.Time) bool`
- `plan:internal/parentreview#closures.ping` -> `func (c *closures) ping(r Round) (shuttleengine.GateResult, error)`
- `plan:internal/parentreview#TestGate_KeepAlive` -> `func TestGate_KeepAlive(t *testing.T)`

**Edit:**
- `internal/parentreview#GateConfig`
- `internal/parentreview#Delivery`
- `internal/parentreview#closures.gate`
- `internal/parentreview/doc.go`
- `internal/shedrecipe#resolveParentReviewClosure`
- `internal/shedrecipe#TestResolveGateSpec_ParentReviewMissingEnv`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
D1: while a delivered request waits with no verdict, the gate pings the idle writer inside its prompt-cache lifetime, at most `maxKeepAlives` times per round, so a reject after a long review resumes the writer on a warm cache.

`GateConfig` gains `KeepAlive time.Duration`, the keep-alive interval, whose zero turns pings off, and `RenderKeepAlive func(ping int) (string, error)`, the render seam beside `RenderDelivery`.
`Delivery` gains `Pings int` (`pings`) and `LastPingAt time.Time` (`last_ping_at`); a `delivery.json` written before this change decodes with both zero.
`Store.RecordPing` bumps the latest round's ping count and stamps its last-ping time under the round lock, touching no prompt count, last-prompt time or waiting notify.

`closures.pingDue` is true when `KeepAlive` is positive, delivery is recorded, fewer than `maxKeepAlives` pings are counted, and `now` is at least `KeepAlive` past the latest of delivered-at, last-prompt and last-ping.
In `closures.gate`, after the expiry, hold and existing carry cases, a due ping returns `closures.ping`: it renders ping number count plus one, records it with `RecordPing`, then returns pending with the text as `Send`.
Its `SendFailedWayForward` says nothing needs doing, the review is still pending and the next keep-alive retries while the round's cap allows; it names no `lyx loom review notify`, which would re-send a notice the reviewer already has.
A carried ping counts toward the cap whether or not its send lands, and the count and clock live in `delivery.json`, so they survive a driver restart.

Bound: a ping is carried only from the open-request-with-no-verdict branch, after the verdict, expiry and liveness-hold checks, so it stops at a verdict, at expiry and at finalize, and never runs while delivery is unrecorded or the reviewer has no live session.
At most one per interval and `maxKeepAlives` per round; after the last the cache is left to expire.
A ping spends no gate prompt and no waiting notify, and `closures.final` never pings.

`doc.go`'s `delivery.json` line names the ping count and last-ping time, and the package doc describes the keep-alive.
`resolveParentReviewClosure` requires `ParentReview.RenderKeepAlive` through `requireSeam` when `KeepAlive` is positive, a wiring guard.

Tests:
`TestGate_KeepAlive`, over `newStore`'s clock and a gate with a one-minute `KeepAlive` and a renderer echoing the ping number, steps through: no ping before the interval since delivery; ping 1 at it; ping 2 one interval after ping 1 with different text; ping 3; no ping after the third.
Further subtests: a fresh gate over the same store resumes the count and clock from `delivery.json` and carries no fourth ping;
no ping before delivery is recorded, while `ReviewerLive` reports not live, after a verdict or after expiry;
`Prompts`, `LastPromptAt` and `WaitingNotifys` are unchanged by a ping;
a waiting notify still carries the delivery prompt ahead of a due ping;
the way forward names no `lyx loom review notify`;
a zero `KeepAlive` never pings.
`TestResolveGateSpec_ParentReviewMissingEnv` gains a `ParentReview.RenderKeepAlive` row with `KeepAlive` set and the seam nil.
`test-budget.yaml`'s `internal/parentreview` row rises by one for the new test; card 5 later deletes five of the package's tests.

**ImpactSummary:** the gate gains one pending-with-send branch behind a zero-off interval, the store one counted write; with KeepAlive zero, as every caller before card 4 sets it, behavior is unchanged.
