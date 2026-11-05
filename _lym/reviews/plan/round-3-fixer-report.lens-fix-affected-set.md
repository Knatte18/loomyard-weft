```yaml
lens: fix-affected-set
outcome: reported
```

Task: rerun each set-premise finding's search (F1, F2, F6, F7) and check the fix covers every member.

**Unchecked findings:** F3, F4 and F5. None of them rests on a set of callers, sites or tests, so there was no search to rerun.

**no findings**

- **F1:** The plan has no other text that ties the cap or its reset to the first read. Card 12's `gave-up` sentence was already updated. Card 04's `RelayState` field list does not depend on the rule.
- **F2:** In the poll, `SetPresence` and `Hub.Beams` are now covered in card 07 at the `away`/`back`/`status` handling, the routing call, the `SetPresence` contract and the rows. Card 10's `stop`, `away` and `back` call `SetPresence` outside the poll, which is outside the finding's set. Card 10's `stop` text still agrees with the changed contract.
- **F6:** Every `ReadWait` contract text in card 03 is reworded: the bullet, and the `creelcli/doc.go` sentence. The resend safety the fix claims for card 08 holds. Card 08 says the handler skips each id the record holds and writes the record after each post. Card 08's statement that a nil return clears the record still agrees.
- **F7:** `setup` and `relay` are the only card 10 verbs whose headings carried argument forms, and both now state a Use string the check admits.

Tool calls:
1. Grepped the plan for cap and reset text tied to "first read". My context did not hold every card.
2. Grepped the plan for `SetPresence`, `Hub.Beams` and `ReadPresence` sites. My context did not hold cards 04 and 11.
3. Grepped the plan for `ReadWait` texts. My context did not hold card 08 or the overview.
4. Grepped card 10's verb headings for Use forms, to rerun F7's search in one call.
5. Read card 08 lines 25-36, to confirm the outbox record makes a second handler run safe. My context lacked card 08.
