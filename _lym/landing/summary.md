# Slack: forward every turn's reply, and stop a late or needless wake

An orch's final reply to an operator's Slack message could be lost, and the next reply could reach Slack about 30 seconds late.
This change fixes both causes and stops the relay waking an orch that its own forward just woke.

## Why

The merged creel Stop command in `claudeengine` ran creel's mail delivery first and exited 0 when it handed mail over.
The turn hook, which forwards the reply to Slack, ran only on the path that recorded a turn end.
So whenever mail waited at turn end, the reply was never forwarded.
An operator's Slack message from presence `terminal` always brings a `[slack presence]` notice right behind it,
so the first reply after switching to Slack was always lost.

The relay's outbound loop also waited out its floor after a long poll that had already waited that long,
so mail posted during the wait was read late.

## What changes

- `claudeengine/settings.go`: `creelStopHookCommand` now reads the hook input once and runs the turn hook first, through `turnHookStep`.
  Creel's delivery follows, then the recording.
  The turn hook sees every turn end, one whose handed-out mail continues the turn included.
  `creelOpenStopHookCommand` reads its own input, since the shared opening moved out of `creelStopDelivery`.
  The `doc.go` files and the `Spec.TurnHookCommand` comment state the new behaviour.
- `slackengine/backoff.go`, `outbound.go`: a long poll that found no mail and lasted the floor or longer is followed by no wait.
  The next poll starts at once, and the wait drops back to the floor (`afterEmptyPoll`).
  A shorter empty poll still backs off as before, so a poll that returns early never runs the loop hot.
- `slackengine/inbound.go`, `relay.go`: the relay remembers each orch it forwarded an operator's message that turned the presence to slack.
  `noticePresenceChange` leaves those orchs out of the `slack` notice, since the message already woke them.
  Notices of other presence changes, and to other orchs, are unchanged.

## Design decisions

- The turn hook runs before creel's delivery rather than after it.
  Delivery exits the shell on success, so any step placed after it would be skipped on exactly the path that lost the reply.
- The turn hook runs in a subshell with its output and errors discarded, so running it first cannot change what the Stop hook prints or exits with.
- A message whose post failed keeps its "turned the presence" mark in `awayPending` until a retry posts it.
  The retry finds the presence already slack, so without the mark the notice skip would be lost.
  A message that posts nothing (empty, over the body limit) drops the mark.

## To check by hand

- Presence-notice skip guards are untested (class design, severity MEDIUM, `internal/slackengine/inbound.go` and `relay.go`).
  The skip itself and the `awayPending` retry path have tests, but the review left the guards around them open.
- Not run: a live Slack session.
  Send a Slack message to an orch while presence is `terminal` and check that the reply reaches Slack, no presence notice wakes the orch,
  and the next orch reply is read by the relay at once.
