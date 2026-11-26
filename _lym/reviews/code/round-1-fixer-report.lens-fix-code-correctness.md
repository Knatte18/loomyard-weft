```yaml
lens: fix-code-correctness
outcome: reported
```

Task: fix-code-correctness check of this round's change set (the F1 doc sentence and the F2 and F3 test rows).

no findings

I traced these by hand and did not run any mutants.

- **F2 row** (`creator "UBOT"`, one message with `User "UBOT"` and `BotID ""`): `authored` returns true only through `isOwnPost`'s user-id clause. With that comparison negated, the bot-id clause is false, and the empty `BotID` makes `authored` return false. The channel is then not adopted and the row fails.
- **F3 row** (`lym back now`): `relayCommandWords` returns two words, `parsePeerCommand` rejects the verb `back`, and the text is forwarded. With `len(words) == 1` removed, `back` runs and nothing is posted to the mailbox, so the row fails.
- **F1 sentence**: the five commands it names match what `inbound.handle` answers rather than forwards.

The fix changed no production code or signature, so no caller is affected.

I made no further tool calls.
