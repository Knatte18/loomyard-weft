# Shuttle start: retry a busy prompt, save its failure, retype a lost one

Two seat start bugs in shuttle's prompt delivery after the skills load: issue #137, and issue #136, which kept only its title.

## A prompt send held busy no longer ends the start at once

`Run.loadSkillsThenPrompt` sent the prompt line once.
When the send's idle wait ran out with the session held, which is a `*SessionBusyError` from a wait that typed nothing, the strand was torn down as died.
Two such starts in a row halt the run as `producer-hard-failure`, which is what issue #137 reports.

The idle wait already polls for the whole `send_ready_timeout_s` window (60 s by default), so a miss on a single poll could not end the start.
In the reported runs, every poll of that window read no input box.
The new `sendStartPrompt` sends the line again with a fresh idle wait, up to `startPromptBusyRetries` (2) more times while the run's deadline has not passed.
Only a `*SessionBusyError` is retried.
`ErrSentTextInDraft` and `ErrForeignBoxText` also wrap `ErrSessionBusy`, but they come after the paste, so they still end the start as before.

## The saved capture now shows the failure

The issue asked to check the idle reading against `startup-capture.txt` in the two run directories.
Those directories are gone, and the file would not have helped anyway:
`abandonStartup` saves `lastStartupCapture`, which is the startup probe's boot screen, taken before the skills load.
`keepFailureCapture` now replaces it with the pane at the failed delivery.
For an engine whose box reading judges a styled capture (`StyledBoxReader`, which Claude implements), it keeps the styled capture with its escapes, since that is the form the box reading parses.
A capture that fails keeps the boot screen.
The exact cause of the missed box in #137 is still unknown.
The next time the box is missed, the saved file holds the screen the reading actually judged.

## A prompt line that started no turn is typed once more

The prompt send counts an empty input box as landed.
If the provider drops the submitted line, the box is empty and no turn starts.
The session then sits idle and ends no turn, so `deliverPromptAtHold`, which runs only at a held turn end, never types the line again.
The strand stays up with no task until its timeout.
This matches #136's title ("starts without the turn naming its prompt file").
An attach after a restart delivers the line once through `DeliverPendingPrompt`, and that delivery can be lost the same way, which matches "also after restart".

Wait now calls `retypeLostPrompt` on every tick.
It types the line again only when all of these hold:

- the line went out through this handle at least the submit window ago (`submit_confirm_timeout_s`);
- no turn start of any instruction lies past the prompt offset;
- the engine parses session signals;
- no pause covers the run.

It does this at most once per handle.
A turn start past the offset ends the watch.
The typing itself goes through the new `typePendingPrompt`, which is the body `DeliverPendingPrompt` already ran.
`typePendingPrompt` judges the prompt's turn again, records `promptSentAt` and skips `DeliverPendingPrompt`'s once-per-handle latch.
So a line delivered at attach can still get its one retype.

## Tests

- `TestStartup_SendFailuresTearDown` gains two checks:
  - a new row where the session reads busy for one whole send window, then idle: the start sends again, types the prompt once and returns the run;
  - the row "a prompt delivery that does not land" now also checks that the saved capture holds the failure screen.
  `startSendEngine` gains `idleWhen` to script the idle reading.
- `TestWait_RetypesAPromptLineThatStartedNoTurn` attaches to a run whose prompt the attach types:
  - with no turn start after it, the line is typed exactly twice;
  - when typing records a turn start, it is typed once.

## Notes for the reviewer

- A Wait test whose run has a deferred prompt line, a fake that records no turn start for it, and a timeout longer than the submit window now sees the line typed a second time.
  The existing held-turn and attach tests use a 10 s timeout, which is shorter than that window.
- The retype relies on the turn-start hook recording the line, the same signal `DeliverPendingPrompt` and the held-turn delivery already rely on.
