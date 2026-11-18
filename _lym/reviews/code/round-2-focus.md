---
round: 2
focus:
  - "internal/shuttleengine/hold.go `retypeLostPrompt`: does the watch for a turn start begin at the events file size read just before the watched typing, so an operator turn recorded before an attach cannot cancel the retype of a dropped line?"
  - "internal/shuttleengine/hold.go `retypeLostPrompt`: is the one-retype latch set only once a retype actually typed, so a retype held with a `*SessionBusyError` before the paste leaves a later tick free to retry?"
  - "internal/shuttleengine/run.go `sendStartPrompt` and the claudeengine `InputBoxText` reader: is the idle layout the board entry describes (ok reply, timing line, top rule, empty caret line, bottom rule, bypass-permissions footer) read as an empty box, and does the landing summary state the outcome?"
  - "internal/shuttleengine tests: does a test drive `Runner.Start` then `Wait` with a dropped prompt line and assert the line is typed twice, and once when the typing records a turn start?"
  - "internal/shuttleengine/run.go busy retry in `sendStartPrompt`: do tests pin the retry cap, the run-deadline stop and the limit to `*SessionBusyError`, including a single paste for a foreign-box reading?"
  - "internal/shuttleengine/run.go and hold.go: are the start resends and the Wait retype cut to the run deadline, idle wait and submit window included?"
  - "internal/shuttleengine/hold.go retype guards (pause cover, `SessionSignalParser` check, submit-window comparison) and `keepFailureCapture` branches in run.go: does each have a test that fails when the guard or branch is removed?"
  - "internal/shuttleengine/wait.go `abandonStartup` doc and the `promptDelivered` field comment in run.go: do they match what the code now does with a failed prompt delivery's capture and with `retypeLostPrompt`?"
---

Each entry restates its site and the question to settle.
