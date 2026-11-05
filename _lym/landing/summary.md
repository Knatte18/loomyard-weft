# creelengine: hold the bell while the turn a ring woke has not ended

A Codex strand with a queued wake no longer gets the `[creel] Mail is waiting for you…` line typed into its composer.
That line started a turn of its own, whose Stop hook handed out the later mail while the queued first mail waited behind it.
In the live creel trial burst case (three mails posted back to back to one idle Codex strand) the queued mail reached the model 17 s after the post, against 8 s for the same burst to the other session.
The cost was one wasted model turn and that delay; no mail was lost or crossed.

## Change

`wakeHookStrand` in `internal/creelengine/deliver.go` now holds the bell for a hook strand while the turn its answered ring woke has not ended.
It reads the woken turn as not ended when the probe has no turn end recorded after the ring.
The hold lasts at most `ringAnswerWindow` (30 s) from the ring.
The queued or running turn ends in a Stop that hands out the mail that arrived since, so that mail needs no bell.

Unchanged: an unanswered ring is still held for the window and followed by the bell at once past it.
A strand with a turn end recorded after the ring gets the bell after the 5 s grace.
With no ring on record the grace rule applies as before.

`doc.go` and the `wakeHookStrand` and `ringAnswerWindow` comments describe the new rule.

## Design decisions

- The hold keys on the missing turn end after the ring, not on a Codex-specific queue signal, so it needs no new state in the store.
- A reading the pane decided carries no turn end time and is taken as it reads, so it holds the bell until the window ends.
  The same applies when the operator interrupts the woken turn before its Stop, which fires no hook.
  In both cases mail that arrived since the ring waits at most 30 s for the bell.
  `doc.go` records this as a bound.

## Tests

`TestScheduler_HookWake` gains three cases:

- No bell within the window while the woken turn has not ended.
- The bell after the grace once a turn end follows the ring.
- No bell within the window when the reading has no turn end time.

## To check by hand

- Not run: the burst case against a live Codex strand in lyt.
  Three mails posted back to back should produce no reminder turn, and the queued mail should reach the model in about the time the other session takes.
