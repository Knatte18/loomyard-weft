# Round 4 fixer-report — discussion

## Changes

- F1, fixed as proposed: D5 now separates the two unrecorded cases.
  A send landed with no turn start seen lets the pause take its entry when the wait ends, and a later turn start runs under the live entry, uninterrupted and unrecorded, bounded by `revive` reporting it as sent with no turn start seen and `lyx tender status` showing it busy under the live entry.
  The turn that ends before the pause's poll keeps its sentence.
  The Acceptance criterion now says the entry is taken after the turn start reads or the wait ends, and records the session when the turn start read within the wait.
- F2, fixed as proposed (first option): the conditioned send also refuses at once, sending no key, on the rewind picker or a lone stray character, and never clears its own unlanded text, reporting it not landed with the text left in the box.
  The conditioned-send bound now names the dropped picker close and clears instead of claiming it only adds refusals.
  Consistency edits beyond the fix key: the report list gains a not-landed outcome, and the refusal Acceptance criterion names the picker and stray character.
- F3, fixed as proposed: the lock-hold bound is one ready reading before typing, then at most one submit window to the landing, then at most one for the turn start; it states the ready reading's shape and that pane captures wait on reed's operation lock and an untimed tmux call, so the windows bound the polling only.
  The concurrent-writer sentence now refers to that hold.
- F4, fixed as proposed (second option): `revive` holds the lock by running inside `pauserecord.Update` with a mutate that changes no entry, keeping `pauserecord`'s import set and accepting the expired-entry prune; the closing bound now says it changes no live entry.

## Disputed

None.

## Deferred

None.
