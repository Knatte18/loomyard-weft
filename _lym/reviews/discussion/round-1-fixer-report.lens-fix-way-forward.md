```yaml
lens: fix-way-forward
outcome: reported
```

I checked every way forward the fix added, and whether each one can be carried out from the state it names. Four findings.

Ways forward checked:
- D1: an envelope left unacknowledged while a live map cannot be read.
- D1: a failed or timed-out hint write drops the connection and reconnects.
- D3: login refuses a mismatched app-level or user token; a mismatched bot token replaces the stored tokens.
- D3: the missing-token notice, plus a post to the prime's channel.
- D3 and D5: an unreadable map fails the guard closed, with a notice.
- D5: refusal on a duplicate shortname: remove the extra clone.
- D5: refusal when another bot created the channel: rename that channel in Slack.
- D6: a refused archive is retried after the outbound wait.
- D8: on `not_in_channel`, invite the bot back.
- D2: a hint outside the map is removed.
- D2: the fswatch fallback.

1. MEDIUM — D3 "When it has the bot token, it posts the same line to the prime's channel the map holds", and the D3 acceptance bullet "posts the same line to the prime's channel, and exits".
   Summary: in the upgrade case F11 targets, the map does not exist yet, so the post has no channel.
   Evidence: D5 makes `.lym/slack/channels.json` new in this task, and nothing in `internal/slackengine` or `internal/hubgeom` names a channel map today (grep is empty). A relay re-exec'd from today's binary has no map. It exits before D5 ensures the prime's channel, and D10 says the old `setup.json` channel key is read by nothing. So the notice reaches the operator only as orch mail, and the bullet can pass only with a map planted by the test. State that the post is made only when the map holds the prime's channel, or name the old setup channel as where it goes, and match the bullet.

2. MEDIUM — the unreadable-map notice: D3 "the hub's orch gets one `[slack relay]` notice naming the map", D5 "refuses the same way (D3)", and D1 "left unacknowledged for Slack's retry".
   Summary: the notice names no way forward, and D1's unacknowledged envelope leads nowhere while the map stays unreadable.
   Evidence: the record's own `PATTERN-refusal-way-forward` constraint requires each new refusal to name its way forward. Nothing fixes or removes the bad file, so Slack's retries run out, and the lost hint is recovered only by a later reconnect. The notice should name how to repair it, for example moving the file aside so the next ensure rewrites it.

3. MEDIUM — D1 "A hint write that fails or passes that limit leaves the envelope unacknowledged, and the holder drops the connection and reconnects".
   Summary: if one hub's inbox stays unwritable (permissions, full disk), every reconnect fails the same way, so one hub stalls push for every hub on the machine.
   Evidence: after a reconnect the holder writes a `resync` hint into every live hub's inbox, which fails again for the broken hub, and each redelivered envelope for that hub drops the connection again. The backoff caps this at one connection per 60 s, but no state is named that ends it, and no notice names the failing inbox. The rule should say what a resync write failure does, and that a persistent write failure sends one notice naming the inbox path, without dropping other hubs' connection.

4. LOW — D5 "When another bot created the channel, the notice names renaming that channel in Slack, which frees the name".
   Summary: after the rename, nothing re-ensures the prime's channel until the relay restarts, and the notice does not say so.
   Evidence: D5 ensures the prime's channel only "at its start". A beam's channel is retried at its next outbound mail, so beams recover on their own. The record also does not say what a relay does after its prime channel is refused at start. The notice should add the restart (`lym slack stop`, then `lym slack start`) for the prime's case.

Out of scope: D8's `not_in_channel` invite through the user token may fail if the operator has left the channel. That fails the same way, then the cap drops the message, but the ensure path's operator invite was not changed by this fix.

Tool calls:
- grep for `channels.json`/`ChannelMap` in `internal/slackengine` and `internal/hubgeom`: to confirm no channel map exists on the upgrade path, which my context did not show.
