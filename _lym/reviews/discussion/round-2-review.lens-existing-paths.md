```yaml
lens: existing-paths
outcome: reported
```

Lens: existing-paths. I traced the existing code paths that meet each state the record introduces or changes.

**Paths traced, per decision**

- D3, per-hub worker start scan (`internal/creelengine/deliver.go:89`, `outboundHubs`) and wake on post (`wakeOutbound`): survives. The record says both skip machine-outbound messages.
- D3, the hour return: **breaks**. The hour is enforced only inside `outboundWorker.run` and `conclude` (`internal/creelengine/outbound.go:269,302`), which D3 cuts machine-outbound messages off from.
- D3, a settle after the message was returned (`settleDelivered` and `settleReturned` do nothing when the message is no longer pending, `outbound.go:127,143`): survives. The record changes this explicitly for machine-outbound messages.
- D3, `WaitReply` and the reply mark reading `state.outbound[id]` (`internal/creelengine/verbs.go:936`, `pool.go:456`): survives. The message stays in the same map.
- D3, the cross-hub confirm lookup by sending id (`internal/creelengine/receive.go:115`): survives. A machine-outbound `To` never equals another hub's receiver.
- D3, the retention carry of pending outbound entries (`internal/creelengine/fold.go:724`), and an older binary's fold of an outbound event: **breaks**. See finding 2.
- D7, the confirmed-read save of an empty outbox (`internal/slackengine/outbound.go:107`): survives. The bridge-post record is a separate file.
- D4 and D8, the existing channel hint going through `readsForHint`, which looks up only the channel map (`internal/slackengine/inbound.go:358-376`): survives. The record says a bridge hint is read through the bridge map.
- D8, the existing `resync` hint from a Socket Mode reconnect (`holder.go` serve and `payOwed`): survives. The bridge poll, at most 60 s, covers the gap.
- D8, `socketHolder.route` dropping an event in an unmapped channel (`internal/slackengine/holder.go:342-355`): survives. The record replaces the drop with a rate-limited bridge resync hint.
- D4 and D8, the operator re-invite after the operator changes (`keeper.go` `inviteOperators`, which iterates the channel map only): **breaks** (LOW). See finding 3.
- D4 and D8, thread replies: the orch-channel path reads threads (`scanThreads`, `ThreadFloor`), and the holder's `hintName` carries a thread: **breaks** (LOW). See finding 4.
- D4 and D8, the `bridge-` prefix test in the resync against orch channel names `<machine>-<shortname>-…` made by the same bot: **breaks** (LOW). See finding 5.
- D4, retirement (`keeper.go` `retire`, which drops the button too) with pending machine-outbound mail of that orch: survives. The message is held `button-off` and returned at its hour, provided finding 1 is fixed.
- D6, `inbound.handle` lower-casing the whole text and the presence-only `status` (`inbound.go:606,653`): survives. D6 states that the app id keeps its case and that `status` gains the app id.
- D8, `retryOrGiveUp`'s single failing key (`inbound.go:702`): survives. The record keeps bridge holds outside it.

**Findings**

1. MEDIUM · design · D3 "Creel owns the hour"
   - Summary: no creel path is named that fires the hour for a machine-outbound message, or re-arms it after a daemon restart.
   - Today the hour is enforced only by the per-hub worker (`internal/creelengine/outbound.go:269` computes the remaining time; `outbound.go:302-303` calls `settleReturned` at the hour).
   - D3 excludes machine-outbound messages from that worker and from the start scan (`internal/creelengine/deliver.go:89`).
   - So with the relay down, which D10 allows, nothing returns the message at its hour, and the `bridge-unsettled` return and its notice never happen.
   - Fix: D3 names the creel mechanism that returns machine-outbound messages at their hour independently of the relay, and how it restarts with the daemon.

2. MEDIUM · design · D3 journal shape, against the existing fold and carry
   - Summary: D3 journals a machine-outbound message "as an outbound message" with a receiving-machine field, which existing readers misread.
   - The retention carry copies a fixed field list for pending outbound entries (`internal/creelengine/fold.go:724-729`: `From`, `To`, `ReceivingHub`, …). It drops the receiving machine and the last hold reason unless they are added.
   - A binary that does not know the field folds the entry as per-hub outbound with an empty `ReceivingHub`. `outboundHubs` (`outbound.go:113-114`) then yields `""`, and a worker starts for it.
   - D8 avoids exactly this for inbound mail with a new event carried under `of`; D3 is silent for outbound.
   - Fix: D3 states a machine-outbound event of its own, carried with `of`, as D8 does for inbound.

3. LOW · design · D4 invites and the bridge map
   - Summary: an operator change re-invites the new operator only into orch channels.
   - `inviteOperators` (`internal/slackengine/keeper.go`, in the span read at 365-385) reads `ChannelMapPath` alone.
   - D4's bridge map records no operator, and the operator is invited only at creation, so after `lym slack setup --operator` the new operator is never added to existing bridge channels.

4. LOW · design · D8 per-message reading
   - Summary: thread replies in a bridge channel are never read or answered.
   - Orch channels scan threads (`inbound.go` `scanThreads` and `readThread`, `ThreadFloor`), and the holder hints thread events.
   - D4 and D8 read bridge channels after the channel cursor only, with "no thread floor". An operator `approve` written as a thread reply, which the Slack UI invites, is silently ignored, and D6 promises an answer in the channel.

5. LOW · design · D8 resync's `bridge-` test
   - Summary: the `bridge-` prefix is not disjoint from orch channel names.
   - Orch channels are `<machine>-<shortname>-…` and made by this bot (`keeper.go` `create`). On a machine whose name is `bridge` or starts with `bridge-`, every orch channel matches the resync's "`bridge-` channel its map lacks, made by this bot" test.
   - Each resync then reads all of them. One whose newest bot post opens with an envelope line, possible because the relay posts orch mail bodies verbatim (`outbound.go:201`, `slackEscape.Replace(message.Body)`), is added to the bridge map as well.
   - Fix: D4 reserves the prefix, for example by refusing a machine name that yields it, or D8 keys the test on the `--` separator as well.

**Tool calls**

- Grep of `internal/creelengine` for outbound-map readers: the manifest held only `outbound.go`, not the fold, start-scan and verbs readers.
- Read of `keeper.go` `inviteOperators` and `inbound.go` `readsForHint`: the manifest named neither span.
- Read of the `fold.go` carry and `moveOrigins`, the `deliver.go` start scan and `verbs.go` `WaitReply`: these were the reader sites the first grep found.
