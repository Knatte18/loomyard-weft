```yaml
lens: way-forward
outcome: reported
```

Task: the way-forward lens. For each refusal, stop, reset and recovery step the cards add, I checked whether the named next step can succeed from the state the refusal leaves.

**Per refusal: the next step, and whether it progresses**

- **Card 4, post or strand reply over the line budget.** Next step: write the text to a file and mail its path. Progresses: the path line is short.
- **Card 4, reply to EXTERNAL over 64 KiB.** Next step: the existing file way forward. Progresses.
- **Card 4, old-binary line over `hookFieldCap` skipped by every claim, flag dropped.** Next step: `lym creel read`, with the message visible in `inbox`. Progresses. D1's Bound accepts that nothing signals it.
- **Card 5, `decision add` refuses an entry whose own notice is over the cap.** Next step: shorten the decision or rationale and put the detail in a file. Progresses, with one gap: the heading (title) is also in the notice and is not named (NIT 3).
- **Card 5, a batch notice split per entry.** Next step: one notice per entry, each fitting by the refusal above. Progresses. Partial-failure reporting is unchanged from today.
- **Card 6, a Stop at the cap claims nothing.** Next step: the turn end is recorded, and the waiter registered by that Stop rings. The set has changed, since each block took a message, and the turn end falls after any earlier ring. The ring resets the count, so the woken Stop claims. Progresses.
- **Card 6, a daemon start holds every strand at the cap.** Next step: a reset — a prompted turn's Stop with `stop_hook_active` false, a PostToolUse claim, or a ring after the waiter reconnects. Progresses. Once `MaxReconnects` is spent, the waiter exits 0 and card 11's bell takes over after the grace.
- **Card 9, a strand with neither field.** Next step: `lym creel read`. Progresses. D6 accepts that nothing signals it.
- **Card 10, `HookWait` from a non-strand.** Next step: "read with lym creel read". Does not progress for EXTERNAL, which `Read` refuses. The text is never shown anyway, since card 13 exits 0 printing nothing (NIT 1).
- **Card 10, a superseded waiter.** Next step: the newer waiter carries the strand. Progresses.
- **Card 10, a dropped waiter connection.** Next step: resume with the same request and re-register. Progresses. Past `MaxReconnects`, it exits 0 and the bell covers.
- **Card 10, the ring.** Next step: the woken turn's first tool call or Stop claims, with the flag standing and the count reset. Progresses.
- **Card 10, the ring hold, including a woken turn the operator interrupted, which records no turn end after the ring.** Next step: card 11's bell, gated on `Idle` and not on `NoTurn`. The bell's prompt turn records a turn end after the ring, which clears the hold. Progresses.
- **Card 11, bell `ErrBusy`.** Next step: the next probe retries. Progresses.
- **Card 11, bell held by a draft, a short pane or a missing box.** Next step: it waits for the stall to clear. Progresses.
- **Card 11, bell `ErrNotLanded`.** Next step: none for that set; the next boundary or the next set. D5 accepts this.
- **Card 11, ring unanswered for 30 s.** Next step: the bell. Progresses.
- **Card 13, hook-read on a Stop fails before the write.** Next step: exit 1, so the turn end is recorded and the mail stays for the next boundary. Progresses.
- **Card 13, ack failure after the write.** Next step: exit 0, and the released claim is handed out again at the next boundary. Progresses, as D2's Bound accepts.
- **Card 13, an input of another event.** Next step: none; it only names the two events. Hook-only; card 14 renders no such entry. Not a finding.
- **Card 13, hook-wait refused, unreachable or superseded.** Next step: exit 0, then the bell after the grace for a marked strand. Progresses.
- **Card 14, the merged Stop's events append fails.** Next step: non-zero exit, as the recording command fails today. Unchanged.
- **Card 15, a receiver without `CreelHooks`.** Next step: `inject --interrupt`, which the same privilege rule admits, or a post without `--interrupt`. Progresses. The text says a hookless engine gets a "typed notice", which holds only for program or EXTERNAL mail of at most 4 KiB; agent mail gets a reminder (NIT 2).
- **Card 15, hold refusal (wrapped).** Next step: post without `--interrupt`, or retry after the hold. Progresses.
- **Card 15, pause lifted before the key (wrapped).** Next step: rerun `post --interrupt`. Progresses: the claim is now uncovered.
- **Card 15, unreadable pause record (not wrapped).** Next step: none. The text says "nothing was interrupted" but not that nothing was posted (LOW 2).
- **Card 15, `interruptTarget`'s own `interrupted` journal append fails after the key.** Next step: none. This error is treated as a refusal "with nothing stored" and hides the stopped turn (MEDIUM 1).
- **Card 15, the store fails after the key.** Next step: post again without `--interrupt`. Progresses once the journal is writable.
- **Card 15, `--interrupt --reply`.** Next step: post `--interrupt` to the sender by name, or `--reply` without `--interrupt`. Progresses.
- **Card 15, `ErrNoRunningTurn`.** Next step: the post is stored. Progresses.
- **Card 16, the `init` check on the trust sentence.** A fail-loud wiring guard; exempt.

**Findings**

1. **MEDIUM — consistency — card 15 (`15-creel-post-interrupt.md`, `Store.post` interrupt path).**
   - D7 says a store that fails after the key answers that the turn was interrupted and the post was not stored. Card 15 applies that to only one of the two store steps after the key.
   - `interruptTarget` sends the key and then journals `eventInterrupted`. If that append fails, it returns `(true, err)` (`internal/creelengine/verbs.go:570-574`).
   - Card 15 treats every `interruptTarget` error as a refusal "with nothing stored" and wraps only the hold and pause-lifted refusals. So this path answers a bare journal error that hides the stopped turn and names no repost step.
   - No `TestStore_Mail` row reaches it.
   - Fix: treat an `interruptTarget` error returned with `true` as a store failure after the key, answering the card's "turn … was interrupted, but the post was not stored … post again without --interrupt" error. Add a row where the interrupt function closes the journal before the `interrupted` append.

2. **LOW — consistency — card 15, unreadable pause refusal.**
   - `interruptTarget`'s `unreadablePauseRefusal` reads "could not read the pause record for X, so nothing was interrupted: &lt;err&gt;" (`verbs.go:583-585`). It names no next step and does not say the post was not stored.
   - Card 15 wraps the hold and pause-lifted refusals but not this one.
   - Fix: wrap it as well, saying nothing was interrupted or posted, with the way forward "post without --interrupt", or retry once the pause record reads.

3. **NIT — consistency — card 15, `CreelHooks` refusal text.**
   - "the receiver gets [the post] by its typed notice when its engine has no hooks" is true only for program or EXTERNAL mail of at most 4 KiB.
   - Agent-strand mail to a pane-mail strand gets a reminder and needs `lym creel read` (`internal/creelengine/reminder.go:27-30`, `isNotice`).
   - Fix: say "by its typed notice or reminder".

4. **NIT — consistency — card 10, `HookWait` refusal.**
   - The way forward "read with lym creel read" cannot succeed for an EXTERNAL caller, which `Read` refuses with `errExternalHoldsNothing` (`verbs.go:131-133`).
   - Card 13 also makes hook-wait swallow the refusal: exit 0, nothing printed.
   - Fix: drop the read way forward, or name it only for a program caller.

5. **NIT — consistency — card 5, notice-too-long refusal.**
   - `decisionWayNoticeTooLong` names shortening the decision or rationale.
   - But `decisionNoticeLine` also inlines the entry's heading (`addedDecisionHeading`, `internal/loomcli/decision.go:280-281`), and `decisionEntries` puts no bound on the title. A long title leaves the named step unable to succeed.
   - Fix: name the title in the way forward too.

Tool calls:
- Grepped `internal/loomcli/decision.go` for notice-failure handling and read `internal/creelengine/verbs.go:560-580`. I needed the exact return of `interruptTarget` after the key and today's notice-failure reporting, which my context showed only in part.
