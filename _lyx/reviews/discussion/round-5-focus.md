---
round: 5
focus:
  - "In D5, check every consequence of `revive` sending to an idle-stalled session while a `pause` waits on the pause record's write lock: for a send that lands with no turn start seen within the wait, does the Decision state what the pause records and interrupts, what a turn start arriving after the pause's interrupt pass runs under, and what bounds that, and does the Acceptance criterion about a pause added during an in-flight `revive` line agree with it?"
  - "In D5, settle whether the stated maximum time `revive` holds the pause record's write lock per send covers everything inside the hold: the ready reading before typing, the submit windows, any clear of unlanded text, and the pane captures that wait on reed's operation lock and a tmux call with no timeout of their own."
  - "In D5, settle whether the claim that `revive` sends no key holds against the keys today's guarded send plays outside its typed text (closing an open rewind picker, clearing a lone stray character, clearing its own unlanded text), and whether the conditioned send's at-once refusals name each one."
  - "In D5, settle whether the Decision chooses how `revive` holds the pause record's write lock without writing, given that `pauserecord` exposes the lock only through `Update`, which rewrites the file and prunes expired resumed entries."
---

Each entry names a site and a question; none caps severity or states a verdict.
