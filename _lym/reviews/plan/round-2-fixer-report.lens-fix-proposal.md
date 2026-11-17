```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals the fixer carried out (fix-proposal lens).

1. **MEDIUM: F3's new refusal case leaves the card 8 and card 9 HoldLost wording wrong.**
   - Changed text: card 2, the FixHold/ClearHold post-push comparison, "Any other read refuses with a `*HoldRefusal` … else naming `lym adopt &lt;subjects&gt;`".
   - Under F3, `FixHold` and `ClearHold` now also refuse when this machine's own hold still stands but differs from what was written. The new `ClearHoldWhoseReplayLostItsRemovalRefuses` row builds that case: a second process on this machine rewrites the hold's tries.
   - Card 8 treats every `*HoldRefusal` as a lost hold.
     - Step 3 reads a `ClearHold` refusal as "another machine took the hold during the check" and sets `HoldLost`.
     - Step 4 reads a `FixHold` refusal as "another machine took the hold during the check, or no hold stands".
     - Card 9 steps 7–8 end a `HoldLost` result with a line that "names that machine or says the hold cleared".
   - For the new case, both are false. The hold still stands and names this machine, and the line says neither.
   - Evidence: card 1's replay rule says "A parent hold origin no longer holds is reported `Lost` and its change is not applied". An own-machine rewrite therefore leaves this machine's hold standing after the push.
   - Card 8 steps 3 and 4 and card 9's HoldLost sentence need to cover a refusal over this machine's own changed hold, or keep it apart from `HoldLost`.

2. **MEDIUM: F4's new row has an input that `holdStands` stops before `movedRun` runs.**
   - Changed text: card 9's `TakeHoldBackFromTheMachineThatTookTheFixTasksRunMovesItAgain`, "the hold names the other machine, `FixMachine` this one …; `TakeOver` of that run is [called]".
   - Card 9 step 2 says `holdStands` re-reads the hold before each of steps 2 to 7, and "once the hold … names another machine it stops … and starts nothing".
   - A take-back runs `WriteAdopt` with `TakeHold`. Card 2 case 6 then rewrites the hold to name this machine. If the hold the row's world returns names the other machine, the adopt stops before any takeover, and `TakeOver` is never called.
   - The row's input should be: the hold after `WriteAdopt` names this machine (take-hold from the other machine), `FixMachine` is this machine, and the origin record holds the other machine's takeover entry.

3. **LOW: card 9's removal sentence names entries card 8 has already removed.**
   - Changed text: card 9, "`testOnlyCodeAllowed` drops every entry cards 2, 3, 5 and 7 added".
   - Card 8 already drops card 2's `FixHold`, `ClearHold` and `HoldRefusal.Error` entries. Read literally, card 9 removes them a second time.
   - The review's fix says "removes every remaining entry". The sentence should say "every remaining entry", or list card 2's `WriteAdopt` alone.

Proposals F2, F5, F6, F7, F8 and F9: I could build no case where they go wrong.
- For F7, `DropLocal`'s body takes no teardown lock, so step 3 does not deadlock under the held lock.
- For F8, card 1's deep-equality replay gives the conflict input the row needs.

Tool calls:
- Grepped `DropLocal` and read `handover.go` lines 430–445, to check whether step 3 or a sibling takes the teardown lock. My context did not hold `DropLocal`'s body.
- Read lines 25–45 of card 1, to check the replay's Lost rule that F3's new rows rest on. My context held only card 1's `fix_machine` line.
