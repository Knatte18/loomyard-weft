# Dobby reminds the operator on Slack of a run waiting on them

A run waiting at PR-Review or Sampler-Session shows nowhere once the operator has left the terminal.
Every dobby, the prime's included, now checks those waits about once a minute and, while presence is `slack`, posts a reminder to `<shortname>:SLACK`.
The line reads `<slug>: waiting at <row> for <n> minutes`, and repeats every `remind_after_min` minutes (default 30) for as long as the wait lasts.

## What changed

- `internal/dobbyengine/waitreminder.go` holds the check, which rides dobby's ticks after the run notices, with no wait of its own.
  It reuses the child and pause reads of `RunNoticeDeps` and adds `WaitReminderDeps` for the rows, the latest resume, the presence, the interval and the post.
- `RunState.OperatorWait` keeps each wait's row and start in dobby's tolerant state record, whatever the presence.
  It clears when the child leaves the row, stops running or a pause covers it.
- A wait starts at the later of the first check that read it and the latest resume of a pause covering the child.
  So a pause and resume dobby never saw, between two checks or while it was down, restarts the wait too.
- `pauserecord.Record.LatestResumed` is the new lookup behind that, and a hub-wide entry counts as covering the slug.
- `internal/beamcli`'s `waitReminderDeps` fills the seams, signing the post as dobby's notices are signed.
- The `remind_after_min` comments now name dobby, not the orch, as the sender; the key's meaning is unchanged, since nothing read it before.

## Design decisions

- The reminded rows are told by the wiring, because `dobbyengine`'s import allowlist bars `loomshed`.
  Beam-Await stays out, since it waits on the beam's members as well as the operator.
- Only runs whose batten state is running with no cancel marker are checked, the same set the run notices read.
  A run batten stopped loses its wait, and its clock starts again when the run goes back to running.
- The last reminder's time is in memory only, so a restarted dobby may remind once early.
  It is forgotten while presence is not `slack`, so a return to `slack` reminds at once of a wait past the interval.
- A failed post is logged at Warn and tried at the next check; creel refuses the post until the relay has read once.
- A child, pause or resume read that fails leaves the recorded wait alone and sends no reminder for that child.
- The Slack geometry resolves at each read, so a hub it fails for loses its reminders and never its dobby.

## Bounds

- While an anchor change stands, no wait is checked.
- A child that leaves the row and returns between two checks, with no pause, keeps its wait.
- Machine sleep counts toward the minutes waited.

## Testing

`internal/dobbyengine/waitreminder_test.go` covers the behavior over fakes, and `TestHubDobbyDeps_WaitReminders` covers the rows, presence, interval and latest-resume seams over a real hub.
`Post`'s real creel call is not exercised by a test: only that the seam is set on both the prime's and a beam's dobby.
