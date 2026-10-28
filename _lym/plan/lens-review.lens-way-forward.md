```yaml
lens: way-forward
outcome: reported
```

Task: way-forward lens review of the creel-agents plan (cards 1–16).

Findings:

1. **BLOCKING — Card 8 / D10: the forward-of-a-chase refusal names a next step that can never succeed.**
   - The refusal's way forward ends with "or forward the message it chases, #&lt;chased&gt;, itself".
   - A chase only exists for a message that is chaseable, which card 7's `isChaseable` defines as read.
   - `Store.Forward` refuses any read message: "message #%d is already read and cannot be forwarded" (`internal/creelengine/verbs.go`, `Store.Forward`, the `msg.Status != StatusUnread` case).
   - The plan's own Shared Decision `former-holder-reply-guard` relies on exactly that refusal.
   - So the second half of the way forward always fails, and only "answer it with lym creel post --reply &lt;id&gt;" progresses.
   - The wording comes from D10, so the fix needs one of two things. Either a parent decision drops the clause, recorded as a `Pattern-Departure` or an `Added after Discussion` entry. Or a card lets `Forward` move a read, chased message.

2. **MEDIUM — Card 10: the `unanswered` envelope cannot carry the message as the card says, and no card makes the edit that would let it.**
   - Card 10 prints "the message under `unanswered`".
   - `daemon.awaitOutcome` sets `resp.Message` only when the outcome is `OutcomeReply` or `OutcomeReturned` (`internal/creelengine/server.go`, `daemon.awaitOutcome`).
   - Neither card 7 nor card 10 lists `internal/creelengine#daemon.awaitOutcome`.
   - As written, the CLI's next step ("read lym creel log --id &lt;id&gt;") still works from the `id` field, but the stated envelope field never arrives.

3. **MEDIUM — Card 7: the unanswered settle step does not say what happens when the sender notice cannot be posted.**
   - `settleUnansweredLocked` journals `unanswered`, posts the sender notice, then answers the waits.
   - A sender strand that has since been torn down fails `checkReceiverLocked` ("unknown receiver", `internal/creelengine/pool.go`) if the notice goes through the ordinary receiver check. The error would then abort the `do` after the `unanswered` record.
   - In that case a registered `--wait-reply` waiter is never answered, and nothing names a recovery.
   - The card should state that a notice to a sender creel no longer lists is skipped or logged, and the waits are answered regardless. A row should cover it.

4. **MEDIUM — Card 12: the parent directive names no step after a refused `lym creel post --parent`.**
   - Card 2 refuses `--parent` when no orch strand is recorded. Creel also refuses a resolved fixed-name orch strand it does not list ("unknown receiver", `internal/creelengine/pool.go`, `Store.checkReceiverLocked`).
   - The new directive text says "Ask it with lym creel post --parent --expect-reply …, then end your turn". It says nothing about a refused post.
   - The driver-notify stencils do cover this case ("record a failed send in the stop report").
   - The parent directive, darn writer and webster Master would end their turn waiting for an answer that cannot come.

5. **LOW — Card 7: the sender's unanswered notice names no next step.**
   - It names "the message and the receiver" only.
   - Card 10's CLI note names `lym creel log --id &lt;id&gt;` and "ask again or decide without the answer".
   - A strand sender woken by the notice gets no equivalent.

6. **LOW — Cards 7/10, observation: a strand sender's `post --wait-reply` that is answered never ends.**
   - `Store.Reply` hands a reply to the waiter only for an EXTERNAL sender. Any other sender's reply goes through `deliverMailLocked` as mail, with no `answerLocked` on `waitReply` (`internal/creelengine/verbs.go`, `Store.Reply`).
   - With the mark, that reply also stops the chases, so the wait never gets `unanswered` either. It ends only at `--timeout`.
   - No stencil uses `--wait-reply`, so no role is stranded today.

Per refusal, the next step and whether it progresses:
- Card 2, `--parent` with a receiver: the named forms. Progresses.
- Card 2, `--parent` with `--reply`: `post --reply` without `--parent`. Progresses.
- Card 2, no orch strand recorded: post by name via `lym reed list`. Progresses.
- Card 6, `--expect-reply` with `--reply`: reply without the flag. Progresses.
- Card 8, former-holder reply guard: `lym creel log --id`. This is a fail-loud guard with no reaching input; the step would progress.
- Card 8, forward of a chase: the reply half progresses; the forward half does not (finding 1).
- Card 10, unanswered exit 1: `log --id`, then ask again. Progresses.
- Card 15, `ErrNoRunningTurn`: `lym creel post &lt;strand&gt; &lt;text&gt;` gives a turn through the ring or a reminder. Progresses.
- Card 15, `refuseInterruptKey`: `lym creel interrupt &lt;strand&gt;`. Progresses for EXTERNAL, programs and orch strands.

Tool calls: none beyond the inherited context.
