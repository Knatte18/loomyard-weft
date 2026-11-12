```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge each review proposal the fixer carried out (F1–F4) by building a case where the proposed change is wrong.

Proposals challenged: F1 (hour timer, take filtering), F2 (unarchive, rename, archive again), F3 (journal events, late settle delivered), F4 (notice marks, cursor moved on approval).

1. **MEDIUM**: the notice-mark home leaves out the send-side holds.
   - **Changed text:** D4, "notice marks: one per kind of relay notice of D4, D7 and D8 …" in the bridge map, and the bridge-notice record "for an unpinned creator or a retired orch".
   - **Summary:** a D7 `peer-unapproved` hold and a D4 `bridge-name-taken` or `bridge-archived` hold come before the pair has any bridge-map entry, and neither falls under the bridge-notice record. So their "one notice per episode" marks have nowhere to live, and a restart repeats those notices.
   - **Evidence:** D4 says the channel enters the bridge map only once it is created or adopted. A name-taken or archived hold means neither happened. D7's `peer-unapproved` hold comes before creation, because creation invites "the approved peer's bot". The new record only covers "a channel the bridge map leaves unmapped, for an unpinned creator or a retired orch (D8)", and the unapproved-peer case has no channel id to key it by. The new D4 criterion "a relay restarted during a hold episode posts no second notice" therefore has no state behind it for these holds.

2. **MEDIUM**: the new bridge-archived way forward can split a pair across two channels.
   - **Changed text:** D4, "unarchive the channel in Slack, rename it to free the name, then archive it again if wanted", and the matching D4 criterion.
   - **Summary:** the case runs in four steps:
     1. The operator unarchives a channel this bot or the approved peer's bot created.
     2. Before the operator renames it, this relay's next try at the bridge-poll ceiling gets `name_taken`. `FindChannel` finds the channel unarchived, and the creator check passes, so the relay adopts and joins it.
     3. The operator then renames it as told. This relay keeps posting to it by id under the new name.
     4. The peer's relay has no entry for it, since D4 drops an archived channel from the map. It computes the original name, finds it free and creates a second channel, so the pair is split.

     For a bot-created channel the right way forward is "unarchive it" alone: the next try adopts it. The rename is needed only when someone else created the channel.
   - **Evidence:** D4: "Otherwise the relay adopts the channel only when its creator is this bot or the approved peer's bot user"; "A post refused because the channel is archived or gone drops the channel from the bridge map, and the next send creates or finds it again per this decision."

3. **LOW**: accepting settle delivered on a settled message opens a double notice.
   - **Changed text:** D3, "Settle delivered is accepted for a returned message, where the per-hub settle leaves a settled message alone."
   - **Summary:** the case runs in three steps:
     1. Creel returns a message at its hour.
     2. The relay's late settle delivered journals `machine-delivered` and posts the "did reach the peer" notice.
     3. The relay crashes before it removes the bridge-post entry. At its next start, D3 has it settle delivered again for every entry it holds.

     The record does not say a second settle delivered on a message already delivered is ignored, so a plan writer can journal it again and post a second notice.
   - **Evidence:** D3: "The relay, at its start and before taking new mail, settles delivered every accepted message its bridge-post record (D7) holds". D7: "An accepted message's entry is removed only once creel has answered its settle".

F1 holds up: I found no case where the scheduler timer or take's past-hour filter breaks a sibling. The return between take and post is still covered by the late-settle notice.

Tool calls: none.
