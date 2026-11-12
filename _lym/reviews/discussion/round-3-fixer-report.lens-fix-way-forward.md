```yaml
lens: fix-way-forward
outcome: reported
```

Lens fix-way-forward: checking that each way forward, step and criterion the fix introduced can be carried out from the state it names and leads to progress.

Ways forward and rows checked:
1. D4 bridge-archived way forward: unarchive, rename, archive again if wanted.
2. D4 criterion: once the operator has unarchived and renamed, the next try creates the channel.
3. D3 hour timer, armed by a machine-outbound post and the start scan; take returns a past-hour message before answering.
4. D3 criteria: return at the hour with no relay running; one `machine-held` record per unchanged reason.
5. D4 notice marks plus the bridge-notice record; D4 criterion: a restart during a hold episode posts no second notice.
6. D8 cursor moving past approved deliveries; D8 approve-then-revoke criterion.

**[MEDIUM]** The restart criterion is unreachable for D4 and D7 holds that occur before the channel exists.
- Changed text: D4's "notice marks: one per kind of relay notice of D4, D7 and D8 … so a restarted relay posts no second notice for an episode". The bridge-notice record "keyed by channel id" for an unpinned creator or a retired orch. The new criterion "D4: a relay restarted during a hold episode posts no second notice for it".
- Evidence:
  - The `bridge-name-taken` and `bridge-archived` holds happen when create fails and the relay does not adopt. The record says "A channel of that name made by anyone else holds the message", and an archived find holds it. In neither case does the pair get a bridge-map entry.
  - D7's `peer-unapproved` hold comes before any create, because the invite needs "the approved peer's bot user". So no map entry exists then either.
  - The bridge-notice record covers only unmapped channels "for an unpinned creator or a retired orch".
  - Result: the marks for these three episodes have no home, and a restart repeats their notices, which contradicts the new criterion.
- The fix should key these marks per pair, by local orch and peer address, in a record that exists before the channel does.

**[LOW]** The archived way forward can split one pair across two channels.
- Changed text: D4's "unarchive the channel in Slack, rename it to free the name, then archive it again if wanted".
- Evidence:
  - After the operator unarchives and before they rename, the relay's next try at the ceiling lists the channel unarchived. When the channel's creator is either bot, D4 says "the relay adopts the channel" by id.
  - The operator's rename then leaves this relay's map holding the renamed channel.
  - The peer relay, lacking a map entry, finds the computed name free and creates a second channel.
  - The steps can be carried out, but they don't reliably lead to one channel per pair.
- Fix: the notice could say that unarchiving alone clears the hold for a channel either bot made, and save the rename for a channel someone else made.

The other items have no findings. The hour timer can be reached with the relay down, because the start scan arms it after a daemon restart. The approve-then-revoke row follows from the cursor rule.

No further tool calls were made.
