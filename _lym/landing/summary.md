# shuttleengine: retry a busy start prompt and retype a dropped one

A seat start no longer dies on one transient "session is not idle" reading, and a prompt line the provider dropped is typed again.
Both fixes sit in the prompt delivery of `internal/shuttleengine`.

Closes #137
Closes #136

## Why

A `burler-review` seat died twice in a row in `shuttle` with `deliver the prompt after loading skills: the session is not idle`.
The session had finished loading its skills and showed an empty `❯` box, yet the idle wait held the send busy and the strand was removed.
One transient miss then cost a whole attempt, and two halted the loom run.

Issue #136 survives only as a title: a review strand starts without the turn that names its prompt file, also after a restart.
The cause in the code path is that the send reads a submitted line as landed once the input box is empty.
A provider can drop a submitted line and leave the box empty, and an idle session ends no turn for a held turn end to type the line at.

## What changes

- A start's prompt send that the idle wait held busy typed nothing, so `sendStartPrompt` sends it again with a fresh wait.
  It does so up to `startPromptBusyRetries` times, only while the run's deadline has not passed.
  Each send's idle wait and submit window now end by that deadline.
- `Wait` calls the new `retypeLostPrompt` each tick.
  It types the deferred prompt line once more when the line went out at least the submit window ago and no turn start lies past the events offset it was typed at.
  It happens at most once per handle, only for an engine that parses session signals, and only with no pause covering the run and the deadline not passed.
  A retype held busy before it typed does not use up the one retype.
- A prompt delivery that still fails tears the start down with the pane at the failure as `startup-capture.txt`, in place of the boot screen.
- `sendThroughPause` keeps the send's deadline, moved later by the pause wait, and its sent-offset target across a resend.
- `DeliverPendingPrompt` now shares `typePendingPrompt` with the retype, which records `promptSentAt` and `promptSendOffset` on the run.

## Design decisions

- The saved captures show the existing box reading already recognizes the `✻ … · done` layout above the `❯` box.
  The fix is therefore the bounded retry, not a change to the idle detection.
  `TestInputBoxText` gains cases pinning that layout, plain and styled.
- The retry covers only a `*SessionBusyError`, which guarantees nothing was typed, so a retry cannot type the line twice.
- The retype needs the turn-start offset recorded just before the line goes out (`sendContext.sentOffset`), so an earlier turn start never reads as the line's.

## To check by hand

- Not run against a live provider: that a seat start survives an idle reading that lags the redrawn prompt.
- The retype after a dropped line was exercised only by the new tests in `startup_test.go` and `attach_test.go`.
