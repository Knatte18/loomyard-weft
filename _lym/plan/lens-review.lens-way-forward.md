```yaml
lens: way-forward
outcome: reported
```

Task: way-forward lens review of the creel-hook-post plan, covering the failure, refusal, reset and recovery ends the cards add.

**BLOCKING — Card 11 vs Card 10 (D4 ring hold).** Card 11's "A newer set clears the mark" wipes out the ring time that card 10's hold depends on, so mail that arrives mid-chain rings the waiter mid-chain.
- Card 10 holds `NoTurn` false "when the mark has a ring time and the answer a non-zero `TurnEndedAt`" until a turn end after the ring.
- Card 11 says, about `wakeMark`, "A newer set clears the mark, so its bell may be typed once more".
- D4's case is new mail arriving during a woken turn's blocked Stop chain. That changes the set, so the mark is cleared and the ring time goes with it.
- The hook reading still shows the pre-ring turn end, because blocked Stops record nothing and a woken turn records no `UserPromptSubmit`. So `NoTurn` reads true and the waiter the last blocked Stop started is rung mid-chain, which D4 forbids.
- Evidence: plan cards 10 and 11 (`plan:internal/creelengine#wakeMark`, `plan:internal/creelengine#worker.wakeHookStrand`); D4 in `_lym/discussion/decision-record.md:87`.
- Fix: a new set resets only the bell and answered state. The last ring time stays until a later ring.

**MEDIUM — Card 15 (D7), pause lifted before the key.** For `post --interrupt`, the reused refusal names a way forward that drops the post.
- `interruptTarget` refuses with `way forward: run "lym creel interrupt <receiver>" again` (`internal/creelengine#Store.interruptTarget`, verbs.go).
- Card 15 has `post --interrupt` call `interruptTarget` and store nothing on its refusals.
- Following that way forward interrupts the turn, but the message is never posted.
- Card 15 should name `lym creel post --interrupt <receiver> <text>` again for this path. It also has no test row that reaches it.

**LOW — Card 15, `--interrupt --reply` refusal.** The way forward "post to the receiver by name" fails when the original sender is EXTERNAL or a program.
- `post --interrupt <name>` then fails `strandTarget` ("unknown strand"), and EXTERNAL is never a receiver.
- Evidence: `internal/creelengine#Store.strandTarget`; creelengine doc's Receivers section ("EXTERNAL is never a receiver").
- The refusal should also name a plain `post --reply <id>` for a sender that is not a strand.

**LOW — Card 15, receiver without the `CreelHooks` mark.** For a pane-mail (Codex) strand, the refusal text is wrong on two counts.
- It says "an interrupted post would not reach it". But card 9 keeps the typed notice for a `PaneMail` strand, so a post does reach it once the stopped turn goes idle.
- Its second way forward says a plain post "waits for the receiver's lym creel read", which is false for that strand.
- Evidence: cards 8, 9 and 15 (`creelengine.Strand.PaneMail`).

**LOW — Card 15, store fails after the key.** The planned reaching test leaves a state where the named way forward cannot succeed.
- The test row removes the receiver from the fake's strands, so the store fails.
- "post again without --interrupt" then fails with an unknown strand.
- The message is right for a journal write failure, but the test pins it only on a path where it cannot progress.
- Evidence: card 15's Tests, last `TestStore_Mail` row.

**LOW — Card 15, refusals it newly surfaces.** The hold refusal tells the caller nothing about what to do with the post.
- `holdRefusal` reads "X is held by Y until T" and names no next step (`internal/creelengine#holdRefusal`).
- For `post --interrupt`, a plain post without `--interrupt` would still store the mail.
- Card 15 states neither this nor whether the post can be retried after the hold.

No further tool calls were made; my context already held everything this lens needed.
