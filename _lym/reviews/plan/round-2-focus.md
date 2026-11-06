---
round: 2
focus:
  - "Card 9 (replies across hubs) adds a reply guard that refuses a delivery whose non-zero ReplyTo matches no outbound entry, before the confirm. Check every test row of card 7 (receive and confirm) that sends a delivery with a non-zero ReplyTo still passes at card 9's gate, and that card 9's Edit list names every test it changes."
  - "Card 8 (outbound worker) maps socket answers to return-notice text bounded to 200 escaped bytes, because CREEL mail is followed as the operator's own. Check each Client.exchange failure path (refusal text, coded refusal, hub-path mismatch, undecodable JSON) has a stated mapping to a creel-written code with no unquoted socket text, and a named test input that reaches it."
  - "Card 9 changes what ComputeReplies counts and what Store.Deliver and Store.storeInboundLocked do. Check every doc, comment and member whose statement the change falsifies sits under Edit, not Uses."
  - "Cards 1, 4 and 5 state a way forward for a corrupt hub record and add the hubs verb. Check each way-forward text is true of every verb and path it names, and that each docs/overview.md list of lym creel verbs is covered by an edit."
---

Focus targets the areas where round 1 found defects; the rest of the plan was reviewed clean.
