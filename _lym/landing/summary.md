# shedadapters: say a timed-out chair was stopped and resume starts over

The halt reason for a timed-out chair said that `lym loom resume` re-attaches the session while it is live.
The seat engine stops a timed-out chair before the step ends, so a resume had no live session to re-attach.
A reader expected the session's context to be recoverable, and the row started over instead.

The reason `MultiLLMProducer` returns for a started chair that timed out now says that the session timed out and was stopped.
Its way forward now says that `lym loom resume` starts the row over in a fresh session.
The elapsed time, limit and last session state are reported as before.
The package doc and the expected strings in `multillm_test.go` follow the new wording.

Closes #281

## Design decisions

- The message changes to match the behavior.
  Leaving a busy timed-out chair live for resume to pick up was the alternative, and it is not taken.
- The time a row spends waiting on its own background work still counts against the chair's limit.

## Merge danger

**Door:** two-way
**Blast radius:** shedadapters
