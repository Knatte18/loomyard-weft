# Dobby reminds the operator on Slack of a run waiting on them

A run that waits on the operator at PR-Review or Sampler-Session used to sit silent.
While presence is `slack`, each worktree's dobby, the prime's included, now posts a reminder line to `<shortname>:SLACK` once the wait reaches `remind_after_min` (default 30), and again every further interval while the wait lasts.
The line names the slug, the row and the minutes waited; the relay carries it to the channel.

## Design

- `internal/dobbyengine` (`waitreminder.go`) checks once a minute, whatever the presence, each running child whose current row is PR-Review or Sampler-Session, and reminds only while presence is `slack`.
  Beam-Await is in `loomshed.OperatorWaitRows` but is not reminded, since it waits on the beam's members as well as the operator.
- The wait's start is the later of two times: when dobby first read the child running at the row with no pause covering it, and the `resumed_at` of the latest resumed entry covering the child in the hub's pause record.
  A pause and resume dobby never observed, between two checks or while it was down, therefore restarts the wait too.
- Dobby keeps the start in its tolerant-JSON state record under the worktree's `.lym/beam/` directory.
  It clears the start when the child leaves the row, stops running, or a pause covers it.
- The last-reminder time is kept in memory, so a restarted dobby may remind once early.
- When presence returns to `slack`, the next check reminds at once of a wait already past the interval.
  A post creel refuses is logged at Warn and tried at the next check.
- `internal/pauserecord` gains `Record.LatestResumed(slug)`, the latest resumed entry covering a slug.
- `internal/beamcli` (`waitReminderDeps`) wires the seams: the rows, the latest resume, the presence and `remind_after_min` through `slackengine`, and the post as creel mail signed `DOBBY` in the dobby's own scope.
  The Slack geometry resolves at each read, so a hub it fails for loses its reminders and never its dobby.
- `internal/slackengine` only rewords the `remind_after_min` comments and docs, which spoke of the orch's reminders.

## To check

- Open finding: the wait reminder's post seam in `internal/dobbyengine` is covered by tests that fake it; the wiring in `waitReminderDeps` (`internal/beamcli/dobbydeps.go`) has an integration test but no end-to-end post through creel was run.
- The `pr-review-wait-silent` note asks for this PR-Review reminder; mark it covered or remove it once this lands.
- Not run by hand: a live reminder to a real Slack channel.
