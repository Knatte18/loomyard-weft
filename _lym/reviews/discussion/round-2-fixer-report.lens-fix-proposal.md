```yaml
lens: fix-proposal
outcome: reported
```

Task: build concrete cases where a reviewer proposal the fixer carried out as proposed is wrong, or breaks something else in the decision record.

**[MEDIUM] F6's new way forward reopens the bypass F6 closes.**
- Changed text: D3, "Its way forward is moving that record aside, after which each hub records itself again at its next `lym creel inbox`."
- Case: the operator moves `hubs.json` aside. `hubregistry.Live` then reads a missing record with no error and answers no hubs, because `readEntries` → `state.ReadJSON` returns an empty file with no error when the file is absent (`internal/hubregistry/hubregistry.go:119-125`). Until every hub has run `lym creel inbox` again, two things go wrong:
  - The holder acknowledges and drops every event of an unrecorded hub, because every live map reads, so no D1 rule leaves them unacknowledged.
  - D3's "held by no other orch's entry in any live hub's map" test passes vacuously for a hub whose map was not listed. That is the exact guard bypass F6 named.
- Nothing then covers the dropped events: no resync is owed, so they wait for a later hint or a restart.

**[MEDIUM] F14's start order contradicts the D3 sentence left above it.**
- Changed text: D3, "A relay that finds a token missing at its start … takes its relay lock and makes its first creel read. It then mails …".
- D3 still says "The relay start, `lym slack start` and `lym slack away` refuse through it", meaning `RequireConfigured`. The code puts that refusal before the lock and the first read: `RunRelay` calls `RequireConfigured` first (`internal/slackengine/relay.go:185-187`), and `runRelay` calls it before building deps (`internal/slackcli/wiring.go:188`).
- The record now asks the relay to both refuse through the shared check and take its lock and read before refusing. Only the second gives F14's order. The sentence was not narrowed to `lym slack start` and `lym slack away`.

**[LOW] F7's retry has no bound and no notice for an invite Slack refuses for good.**
- Changed text: D5, "Each ensure of a channel the map holds invites the setup's operator when the entry records another operator or none, until the invite stands."
- Case: the operator's account is deactivated, or the invite is refused with a code that never clears. Every ensure then repeats the invite, and nothing mails the orch.
- If a failed invite counts as a prime's channel that does not stand, the existing "ensured again at each wake from the outbound read's wait until it stands" repeats a failing Slack call on every outbound wait. Nothing tells the orch the operator is outside its channel.

**[LOW] F3 changes what `last_slack_ok` means.**
- Changed text: D1, the last-healthy stamp at the end of each outbound read for every relay, which `lym slack status` reads.
- Case: a non-holder relay on a broken bot token. It makes no successful Slack call, yet every outbound read stamps the field.
- `status` still prints the stamp as `last_slack_ok` (`internal/slackcli/cli.go:533-534`). It now reports Slack health for a relay that has not reached Slack. The record neither renames the status field nor says it now means liveness.

**[LOW] F9 lets one hub's `lym slack login` end other hubs' relay processes, with no stated route.**
- Changed text: D3, "A login that changes a stored token ends every live relay of the machine's live hubs".
- `PATTERN-lym-files` in Constraints lets only the relay reach other hubs' files, through the live-hubs seam. Nothing names how a CLI verb in one hub finds and ends another hub's relay. Those relays are owned through each hub's `runproc.HubOwner` record (`PATTERN-child-processes`), so the step cannot be carried out as the record now stands.

Tool calls:
- `grep` of `readEntries` in `internal/hubregistry/hubregistry.go`, to see how `Live` treats a missing hubs record; my context showed only `Live` itself.
