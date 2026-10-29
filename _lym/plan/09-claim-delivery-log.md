# Card 9 — claim-delivery-log

**Edit:**
- `internal/creelengine#Reading`
- `internal/creelengine#Reading.Ack`
- `internal/creelengine#Reading.Release`
- `internal/creelengine#Store.ClaimAtBoundary`
- `internal/creelengine#Store.ringHook`
- `internal/creelengine#worker.typeBell`
- `internal/creelengine#daemon.handOut`
- `plan:internal/creelengine#daemon.awaitAck`
- `internal/creelengine/doc.go`

**Create:**
- `plan:internal/creelengine#Reading.logOutcome` -> `func (r *Reading) logOutcome(outcome string, released, dropped []int64, err error)`
- `plan:internal/creelengine_test#TestStore_ClaimLog` -> `func TestStore_ClaimLog(t *testing.T)`

**Uses:**
- `plan:internal/creelengine#Store.FailJournalWrites`
- `internal/creelengine_test#env.start`
- `internal/creelengine_test#awaitOutcome`
- `internal/creelengine_test#env.probeWaits`
- `internal/creelengine_test#env.tick`
- `internal/creelengine_test#env.typed`
- `internal/testkit/logcapture#Capture`
- `internal/logger#SetVerbosity`

**Intent:**
Decision D5: the daemon logs each hook delivery attempt and each bell send through `internal/logger` to the daemon log.

The struct `Reading` gains the unexported `path string`, `tool`, `stop` or `ring`, set by `Store.ClaimAtBoundary` from the boundary and by `Store.ringHook`; a read's reading has none and logs nothing.
`Reading.logOutcome` writes one line, message `creel claim`, with the attributes `strand`, `path`, `ids` (the message ids handed out), `echo_ids` (the echo items' ids handed out) and `outcome`:
- `acknowledged` at Info from `Reading.Ack` on success, a confirmation lost after the journaled read included;
- `ack-failed` at Warn from `Reading.Ack` on a failed journal write, adding `error`, `released_ids` (the ids card 3-ack-failure-releases released) and `dropped_echo_ids` (the echo items card 7-ring-echo dropped; a ring's line lists none, since a ring hands out none);
- `released` at Info from `Reading.Release`, which every end before the acknowledgement reaches (the client dropping or failing, a write error, a cancelled ring wait), logged only when the reading held ids or echo items, so a released empty claim keeps its Debug `nothing` line as its one line.
`daemon.awaitAck`, which card 4-ring-claims-mail gave the acknowledgement half of `daemon.handOut` with its `could not journal an acknowledged read` Warn, keeps that Warn for a read's reading alone, the one card 3-ack-failure-releases marks with `read`, so a claim's or a ring's journal failure logs once, as `ack-failed`.
A claim that hands out nothing logs at Debug from `ClaimAtBoundary`, `outcome` `nothing` and `reason` one of `paused`, `held` (a live hold from another sender while the strand holds unread mail none of which is the owner's), `stop-cap`, `stop-count-unknown` or `nothing-claimable`, since a standing flag runs the claim at every tool call (`PATTERN-spawn-observability`'s Level policy); `ringHook`'s ring of a marked waiter that a pause or an unreadable pause record keeps from claiming logs the same Debug line with path `ring` and reason `paused`; a set that changed before the locked step claims nothing and logs nothing, since the next probe decides afresh.
`worker.typeBell` logs `creel bell` at Info for every send it makes, with `strand`, `ids` (the claimable ids) and `outcome` `sent`, `not-landed`, `busy` or `failed`, adding `error` on the last.
A hook that never reaches the daemon is not logged by it (D5's Bound).

Test (tier0): `TestStore_ClaimLog`, a table that is not `t.Parallel` because it redirects the process-global logger (`logcapture.Capture`, then `logger.SetVerbosity(2)` to raise it to Debug, which Capture's cleanup resets), its bell and ring rows driving the scheduler with `env.probeWaits`, `env.tick` and `env.typed` as `TestScheduler_HookWake` does, each row asserting exactly one `creel claim` or `creel bell` line with its level, path, ids, echo ids, outcome and reason:
tool-boundary claim acknowledged; Stop claim acknowledged; claim released; an empty claim released, logging the Debug `nothing` line alone; claim ack-failed under `FailJournalWrites(0)` for a claim that hands out an owed echo and two messages, naming the released ids and the dropped echo ids; ring ack-failed under `FailJournalWrites(0)`, naming the released ids and no echo ids; a claim handing out an echo after an acked ring, acknowledged with `echo_ids`; ring acknowledged, ring released and a paused ring (path `ring`, reason `paused`), through a scheduler with a marked `HookWait`; nothing handed out for each of `paused`, `held`, `stop-cap`, `stop-count-unknown` and `nothing-claimable`; a bell sent after the grace with no waiter, a bell whose send did not land (`SetSendError(ErrNotLanded)`), a busy bell (`ErrBusy`) and a failed one (another error, its `error` attribute named).

`internal/creelengine/doc.go` states the log lines, their levels and the Bound, and that the `acknowledged` line records the ids D3's loss window loses as read.

**ImpactSummary:** Log lines only, added inside creelengine's claim, ring and bell paths; no signature or behaviour change, and handOut's existing Warn narrows to the read verb.
