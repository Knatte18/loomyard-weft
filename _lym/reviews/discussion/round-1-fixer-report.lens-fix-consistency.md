```yaml
lens: fix-consistency
outcome: reported
```

Fix-consistency lens: checking that the changed text agrees with its sibling decisions, criteria and cross-references.

1. **MEDIUM.** The D8 resync text contradicts itself between the F5 change and the F7 change.
   - Changed text: D8 says "It considers only messages whose bot is this bot or whose app is the pinned app of their envelope's `from` machine". A few lines later it says "A `bridge-` channel whose creator bot's app no pin holds is left unmapped, and when its newest envelope names this hub's shortname at either end, the resync posts one notice…", and the notice names "the envelope's `from` machine".
   - Problem: on first contact the only envelope messages come from the unpinned creator's bot. The "considers only" rule excludes exactly those messages, so the notice's condition can never hold and the notice has no `from` machine to name. The F7 path is unreachable as written.
   - Fix: limit "considers only" to choosing the local orch. Alternatively, state that the notice reads the unpinned creator's newest envelope for routing only.

2. **LOW.** D6's opening rule now disagrees with the new bridge-channel rule.
   - Changed text: D6 adds "In a bridge channel, a relay handles `approve` and `revoke` only when they name that channel's peer machine… Any other operator text there is answered once… as not delivered".
   - Problem: the unchanged sentence just above still says the commands are "handled like `away`, `back` and `status` in any channel the relay reads". Under the new rule, `away`, `back` and `status` in a bridge channel get the not-delivered answer instead.
   - Fix: limit that sentence's "any channel" to orch channels, or name the bridge-channel exception there.

3. **LOW.** The D8 Bound conflicts with the new F7 resync read.
   - Changed text: D8's resync now reads an unpinned-creator `bridge-` channel to find its newest envelope for the notice.
   - Problem: the unchanged Bound in the same decision says "a channel no approved bot made is never read".
   - Fix: change the Bound to "never delivered from", or to "read only by a resync for its notice".

4. **LOW.** Two unchanged references still point at the old wording.
   - The auto-mode assumption "An operator message in a bridge channel is not mail (D8)" now refers to D8, but the rule moved into D6 ("handled as D6 states for a bridge channel").
   - D8's "A relay also resyncs at its start and at each bridge-poll ceiling" doesn't say whether that is the new bridge resync or the existing resync, which rereads every orch channel. The change separated those two hints, so this sentence is now ambiguous.

No further tool calls made.
