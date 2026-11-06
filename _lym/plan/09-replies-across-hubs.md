# Card 9 — replies-across-hubs

**Edit:**
- `internal/creelengine#Store.Reply`
- `internal/creelengine#Store.Forward`
- `internal/creelengine#Store.releaseGoneHolder`
- `internal/creelengine#Store.Deliver`
- `internal/creelengine#Store.storeInboundLocked`
- `internal/creelcli#forwardCmd`
- `internal/reportengine#ComputeReplies`
- `internal/reportengine#RoleReplies`
- `internal/reportengine/doc.go`
- `internal/creelengine/doc.go`
- `internal/creelengine_test#TestStore_Mail`
- `internal/creelengine_test#TestSweep_RemovedHolders`
- `internal/creelengine_test#TestScheduler_Chases`
- `internal/creelengine_test#TestScheduler_Outbound`
- `internal/creelengine_test#TestDaemon_CrossHubDeliver`
- `internal/creelcli#TestCreelCLI`
- `internal/creelcli#rig.startDaemon`
- `internal/reportengine#TestComputeReplies`

**Extends:**
- `internal/creelcli/doc.go`

**Uses:**
- `internal/testkit/creelfake#Hubs`
- `internal/creelengine#Store.checkReceiverLocked`
- `internal/creelcli#daemonHubs`
- `internal/creelcli#rig.addHub`

**Intent:** Carry replies back over the same route, and settle the local rules of a message from another hub (D4).

`Store.Reply`: past the holder and chase checks, an `original` with a `SendingHub` is answered by an outbound message and no local post.
It validates with `validateStampedLine`, so a reply whose stamped line would pass 8 KiB is refused at `--reply`, and journals `eventOutbound` to `original.From` with `ReceivingHub` `original.SendingHub`, `ReplyTo` `original.SendingID` and `Answers` the local id, which card 3's fold marks replied, so the chase stops when the reply is journaled.
It then signals the worker.
The hub path is the one D1 resolved at the message's arrival, so `Reply` makes no call under its lock.
A reply that returns undelivered leaves the original marked replied, and D2's notice tells the replier, who may reply again.

`Store.Deliver`, after the duplicate check of step 3 and with its receiver rules: a `ReplyTo` not 0 is taken as a reply only when `outbound[ReplyTo]` exists, its `ReceivingHub` is the sending hub D1 resolved and its sender is the delivery's receiver.
Anything else is `refused` with a reason naming the id, whether it never went to that hub or retention dropped it.
The input that reaches this past the upstream checks is a well-formed `deliver` from a resolved hub that names a local id it was never sent, which the row below writes.
This guard stands before the confirm, so it would refuse the reply-to case of card 7's row "A `deliver` of a real pending id differing in each of sender, receiver, reply-to, mark and post time is refused as a drop" with no drop.
That case's `ReplyTo` now names a real `outbound` entry of the receiving hub, sent to the sending hub by the delivery's receiver, so the delivery passes this guard and the confirm answers no.
On storing, `storeInboundLocked` keeps kind reply, and `Deliver` answers `waitKey{waitReply, ReplyTo}` with `OutcomeReply` and the stored message, so every wait ends with `from` the bare name beside the trust fields; the fold marks the outbound entry replied, and the reply still reaches the sender as mail.

`Store.Forward`: a message with a `SendingHub` forwarded to a program or to a name of another hub is refused with `creel: message #<id> came from another hub and is forwarded only to a strand of this hub; way forward: post it anew with lym creel post <receiver> "<text>"`.
A forward to a strand of this hub stays as today, and the checks run before `checkReceiverLocked`, so such a name gets this refusal and not an unknown-receiver one.
`forwardCmd`'s `Long` and `internal/creelcli/doc.go` state the rule.

`ComputeReplies` counts the mail this hub's roles got across hubs as it counts a post: `inbound` reads as `posted`, and an `outbound` entry whose `Answers` names a marked message counts `Replied` for its sender.
An `outbound` post and an inbound reply count nothing, since the report counts what this hub's receivers were asked, and chases of such mail then have their `Expected`.
The Replies section of `internal/reportengine/doc.go`, the `RoleReplies` field comments and `ComputeReplies`' doc comment name the `inbound` and `outbound` counts, the last also that an `outbound` entry counts by its `Answers`.

`Store.releaseGoneHolder`: when the step back is the sender of a message with a `SendingHub` and an empty `Path`, the message is journaled undelivered, as for a step back that is no strand, since `isStrandName` alone would return it to a strand this hub does not hold.
`noticeUnansweredLocked` needs no change: such a sender is listed nowhere here, so a settle sends no notice to the other hub.

`internal/creelengine/doc.go` drops `sending across hubs` from its "Later and not planned" line, keeps the MCP layer there, and documents replies; the cards before it wrote D1 to D6's sections.

Rows:
- `TestDaemon_CrossHubDeliver`: the reply-to case of the drop row writes that `outbound` entry and its `remote-delivered` record as journal lines before `startReceiving`, so no worker starts for it, and it still asserts a drop and a notice.
- `TestStore_Mail`: a reply to a message from another hub journals an outbound message and marks the original replied; a reply by a local strand it was forwarded to does the same; an over-cap stamped reply is refused; a forward to a program and to `zz:orch` is refused naming a new post; a forward to a local strand works.
- `TestScheduler_Chases`: the replier's chase stops when the reply is journaled; a reply to the chase of a cross-hub message answers the chased message through the outbound route; a settle of a cross-hub message journals `unanswered` and posts nothing to the other hub.
- `TestSweep_RemovedHolders`: a gone holder's unread message from another hub is journaled undelivered.
- `TestScheduler_Outbound`: a reply from the holder reaches the original sender as mail, ends its `--wait-reply` and marks the original replied; a delivered reply naming an id that did not go to the replier's hub is refused; one naming an outbound id whose sender is not the delivery's receiver is refused; a reply after the sending side's retention dropped its message is refused.
- `TestCreelCLI`, tier1 and end to end, over `newRig` and `rig.addHub`, with `rig.startDaemon` passing `Hubs` from `daemonHubs`, so the real `hubDirectory` serves both daemons from one `t.TempDir()` record: a post from a strand reaches the other hub's strand, whose `read` shows the stamp, whose `post --reply` ends the sender's `post --wait-reply`, and whose reply shows `from` the bare name beside the `trust` and `sending_hub` fields.
- `TestComputeReplies`: a marked `inbound` entry counts `Expected`, its chase and unanswered entries count under the same role, an `outbound` entry answering it counts `Replied`, and an `outbound` post counts nothing.

**ImpactSummary:** `Reply`, `Forward` and `releaseGoneHolder` change only for a message with a `SendingHub`, so every local path and its tests hold; no signature changes, and `ComputeReplies` adds two event cases, which `lym report replies` reads with no change.
