---
round: 2
exclude_lenses: []
focus:
  - "Bridge channel read lifecycle (decision-record.md D4 name_taken adoption and D8 resync add): the existing Slack adopt in internal/slackengine/keeper.go starts a channel's read cursor and thread floor at the newest message. Settle whether the record states where a bridge channel's cursor starts when it is adopted after the peer posted, or added by resync, and whether the first message in such a channel is delivered once."
  - "Settle and outbox durability (D3, D7, D10): internal/slackengine/outbound.go saves an empty outbox record on every confirmed read of SLACK-mailbox orch mail, and D10 lets the relay stop when no button is on, so it can stay down past creel's outbound deadline in internal/creelengine/outbound.go. Settle whether a bridge message Slack accepted survives orch-mail reads and a relay restart, and what creel does with a settle-delivered for a message it already returned."
  - "Operator commands and channel trust (D6, D8): both machines' relays read every bridge channel, command handling in internal/slackengine/inbound.go is gated on the operator alone and lower-cases the whole text, and bridge channels are public. Settle what each relay does with an operator command or plain text in a bridge channel, whether approve can pin a machine's own app or revoke act on both machines, whether an app id keeps its case, and whose envelope messages resync adoption trusts."
  - "Event routing and first contact (D8): internal/slackengine/holder.go drops an event in a channel no map holds, and the existing resync hint rereads every open orch channel. Settle what an event in a channel outside every hub's map can trigger, at what rate, and what the receiving operator is told when a channel was made by a bot whose app no pin holds."
  - "Acceptance criteria against Decisions: settle whether every behaviour a Decision states, among them the return after repeated Slack refusals, receipt under a former machine name, the answer to non-command operator text in an orch channel, and the bridge-name-taken hold, has a checkable criterion, and whether each criterion checks a stated Decision."
---

No lens is excluded: each of the four lenses that ran raised at least one MEDIUM or worse finding or sits in an area still being rewritten.
