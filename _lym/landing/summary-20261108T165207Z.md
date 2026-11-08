# Slack: forward a reply when mail waits at turn end, and forward at once

An orch's final reply sometimes never reached Slack, and a forward that did could arrive half a minute late.
This change fixes both, and drops a presence notice that woke the orch for nothing.

## The reply dropped when mail waits at turn end

In a hub, an orch's Stop hook is one merged command (`creelStopHookCommand` in `internal/shuttleengine/claudeengine/settings.go`).
It ran creel's turn-end delivery first, and when `lym creel hook-read claude` handed out mail, the command exited 0 at once.
The turn hook, `lym slack hook claude`, ran only on the path that records a turn end, so that turn's reply was never posted to the relay.
The first reply after the operator writes from Slack always hit this: the message flips the presence to slack, and the relay's `[slack presence]` notice waits behind it at the turn's end.

The merged command now pipes the hook input to the turn hook first, then delivers creel mail, then records.
The turn hook's output is still discarded, so it never reaches the hook's output, and a Stop that hands out mail still records nothing.
`creelStopDelivery` no longer reads the input itself; each Stop command opens with `in=$(cat)`.

## The late forward

The relay's outbound loop long-polls its creel mailbox for up to sixty seconds.
After a poll that timed out empty, it waited a further 10, 20, 40, then 60 seconds before polling again, and mail posted during that wait sat unread.
The new `loopWait.afterEmptyPoll` treats a poll that lasted at least the floor as the wait itself, so the next poll starts at once.
A poll that comes back empty sooner than the floor still backs off as before, so the loop cannot run hot.
The five-second wait after a read that found mail, and the backoff after a failure or a rate limit, are unchanged.

## The redundant presence notice

When an operator's Slack message turns the presence to slack, the orch it is forwarded to already knows the operator is in Slack.
The inbound read now keeps each orch whose forward stood after such a turn, and the wake loop takes that set when it mails the notice.
A notice of slack leaves those orchs out; every other orch of the hub still gets it, and a notice of terminal goes to every orch as before.

## Design decisions

- The turn hook runs before creel's delivery rather than after it, so it runs whatever hook-read does, including when it fails.
- A forward that failed and is retried does not count, so the orch gets the notice as well as the message, not neither.
- The set of orchs is cleared at every wake, whatever the presence, so a stale entry cannot suppress a later notice.

## Tests

- `TestCreelStopHook_RecordsATurnEndOnlyWithoutABlock` runs the command under `sh` with a stub `lym`.
  New rows check that the turn hook gets the payload when mail is handed out and when the turn ends.
- `TestBuildSettings_TurnHookCommand` pins the new order of the merged command.
- `TestOutboundRead` adds a case where an empty poll lasting its timeout returns no wait.
  The existing backoff case covers a poll that returns empty at once.
- `TestRunRelay` adds a case where an operator's message turns the presence to slack.
  It checks that only the beam's orch gets the notice, not the orch the message went to.
