```yaml
lens: bound-breaker
outcome: reported
```

Bound-breaker lens: I attacked the record's claims, bounds and ordering promises with concrete counterexamples.

**Findings**

1. **BLOCKING — D4 / D5 (and D1 routing).**
   - Summary: the claim that the shortname "is unique among a machine's hubs" is false, and D5's "adopt a live channel the bot is a member of" then lets two hubs relay one channel.
   - Evidence: `hubregistry.Record` and `Live` keep and list several hubs that read the same shortname. `Lookup` has an explicit refusal for that case: "mail across hubs needs distinct shortnames, but %d hubs on this machine read the shortname…".
   - Counterexample: two clones on `neve` both read `ly`. Hub B's create of `ly-neve-prime` gets `name_taken`. The channel is live and the bot (shared by every hub on the machine) is a member, so B adopts A's channel. Both maps now hold one channel id, and the holder's routing sends both orchs' traffic to whichever hub it reads first.
   - The same adopt path breaks names inside one hub:
     - A beam whose slug is literally `beam-prime` and a beam whose slug is `prime` both map to `ly-neve-beam-prime`.
     - A 6-hex hash truncation can collide.
   - Either way, the second orch adopts the first's live channel. Adopting is safe only for a channel no other map on the machine holds.

2. **BLOCKING — D1 (trust boundary).**
   - Summary: the inbox is a plain file directory, and everything in it is forwarded with operator authority. Any process of the user can plant an "operator" message for any hub's orch, with no Slack check.
   - Evidence: today the relay forwards only what it reads from `conversations.history`, which Slack authenticates as `User == OperatorID` (`inbound.handle`). Under D1 the hub relay trusts whatever event file sits in `.lym/slack/inbox/`.
   - Counterexample: an agent in hub A writes a fake event file into hub B's inbox. B's orch receives `[slack] …` from `ly:SLACK` as the operator's word. That bypasses creel's cross-hub confirm and `trusted_hubs` rule (`PATTERN-agent-addressing`).
   - D1's stated bound ("only events whose channel a live hub's map names") limits routing, not authenticity.
   - Possible fix: the inbox carries only a wake hint (channel, ts), and the relay fetches and checks the message from Slack itself.

3. **BLOCKING — D1 / Goal (liveness).**
   - Summary: nothing detects a dead socket, so with no poll, messages silently stop.
   - Evidence: the record relies on Slack's pings and claims "no wake loop of its own while idle". It names only a `disconnect` envelope or a read error as reconnect triggers.
   - Counterexample: the machine suspends and resumes, or the network changes. The TCP connection is half-open, the read blocks with no error, and Linux TCP keepalive takes about two hours to notice. Every operator message in that window is never pushed, and no catch-up runs, because there is no start and no new connection.
   - This needs a read deadline or ping watchdog, which is a wake the record says it does not have.

4. **MEDIUM — D2 with D1 (cursor skip).**
   - Summary: the "skip ts ≤ cursor" rule loses a message whose envelope Slack retries.
   - Counterexample: the inbox write for message ts=4 fails transiently, so its envelope stays unacknowledged as D1 intends. The write for ts=5 succeeds and is handled, and the cursor moves to 5. Slack's retry of ts=4 arrives and D2 skips it as not past the cursor. It is lost, and no catch-up runs, because there was no reconnect.

5. **MEDIUM — D2 with D7 (threads).**
   - Summary: the claim "a catch-up there covers every gap" fails for the thread replies D7 now forwards.
   - Evidence: `conversations.history` returns top-level messages only; thread replies need `conversations.replies`.
   - Counterexample: the operator replies in a thread during a lock hand-over or reconnect gap. Catch-up never sees the reply.

6. **MEDIUM — D6 (stop ordering).**
   - Summary: "`lym slack stop` archives every open beam channel before it ends the relay" races with the running relay.
   - Counterexample: stop archives `ly-neve-x`. Before the relay ends, the relay's outbound read posts mail from `ly:x:orch`, gets `is_archived`, and D8 reopens the channel. The relay then ends with the channel open, against the decision.
   - Ending the relay first and archiving after closes the race.

7. **MEDIUM — D2 (ordering key).**
   - Summary: "the inbox is handled in write order" names no order key that survives a lock hand-over.
   - Counterexample: with a holder-local counter or a per-process name, the new holder's resync marker can sort before files the old holder wrote, or interleave with them. An mtime order is not reliable at file-system granularity either. The marker-before-events promise needs a machine-global, monotonic file name.

8. **MEDIUM — D6 (archive trigger).**
   - Summary: "archives every beam orch … that `slackengine.Hub` no longer lists" has no guard for a failed or transient read.
   - Counterexample: `Hub.Beams` errors, or reads an empty list mid-rewrite of the board or fabric. The relay archives every beam channel, then reopens each on its next post with the user token: churn, and channels left archived while their orchs are live.
   - The trigger should need a successful read, and it applies "listed", not "live". A beam whose orch was stopped by `orchcli.BeamDown` but whose pair remains stays listed, so its channel is never archived. That conflicts with D5's "live beam orch" wording.

9. **MEDIUM — D5 (presence trigger).**
   - Summary: "ensures every live beam orch's channel when presence becomes `slack`" has no detecting mechanism now that the poll is gone.
   - Evidence: today `inbound.presenceChanged` runs on every poll pass. D1 and D2 name only an fswatch on the inbox. `lym slack away` writes `presence.json` from the CLI, and the relay notices only at its next outbound wake, up to about a minute, or not at all if nothing reads presence there.

10. **LOW — D7 (internal contradiction).**
    - Summary: "A message … with any other subtype is skipped" contradicts forwarding file messages.
    - Evidence: Slack delivers a file upload as subtype `file_share`, so under the stated skip rule the files bullet never fires. `file_share`, and `thread_broadcast` if wanted, must be excepted.

11. **LOW — D5 step 4 with D1 (creation window).**
    - Summary: a message written in a new channel before its map entry exists is lost.
    - Counterexample: the operator is invited at step 3 and writes at once. The holder finds no map entry, so it acknowledges and drops the event. Step 4 then sets the cursor at the channel's newest message, which is the operator's, so catch-up skips it too.

12. **LOW — D3 (upgrade).**
    - Summary: existing machines hold `slack.json` as `{"token": …}`; the record names neither that key's reading nor the upgrade path.
    - Evidence: `credentials.go`'s `tokenRecord` has `Token` with the JSON key `token`.
    - Counterexample: after the binary update, `RequireConfigured` refuses every relay start, including dobby's `EnsureRelayWhileAway` restarts, until the operator runs `login` twice more. This is not stated as a migration step.

13. **LOW — D3 / D10 (unverified scope).**
    - Summary: the claim that the user token can invite the bot with only the user scope `channels:write` is not checked against `conversations.invite`'s user-token scope list. The research covered bot scopes only (`channels:manage`, `channels:write.invites`, …).
    - If the user-scope list differs, reopen fails with `missing_scope`, and every reopen in D6 and D8 breaks.

14. **LOW — D1 (reconnect backoff).**
    - Summary: the "failed open backs off one to sixty seconds" bound does not cover an open that succeeds and then drops at once (for example `link_disabled`, or a revoked app). The base resets each time, so the relay loops at one-second reconnects without end.

**Claims attacked**
- D1 one connection per machine by lock: held across hand-over; broken for liveness (3).
- D1 acknowledgement after the write stands: held, but it feeds 4.
- D1 routing bound: broken for authenticity (2) and for duplicate maps (1).
- D2 catch-up covers every gap: broken (4, 5); write-order promise underspecified (7).
- D3 user token used only for two calls on mapped channels: held; scope unverified (13); upgrade gap (12).
- D4 shortname unique per machine: broken (1); 80-character cap: held, but the hash can collide (1).
- D5 create, `name_taken` and adopt: broken (1); presence trigger has no mechanism (9); creation window (11).
- D6 archive within one wake: held, roughly a minute through `ReadWait`'s timeout; unguarded read (8); stop race (6). Prime never archived: held. Crash archives nothing: held, since other `runproc` End paths also skip the archive, consistently.
- D7 subtype rule: contradiction (10). Presence commands in any channel: held.
- D8 refusal-cap bound: held, bounded by the cap of five. Gone-beam mail to the prime's channel: held if the listing check comes before ensure.
- D9 presence per hub, compatible with `slack-presence-auto`: held.
- D10 old setup file read tolerantly: held, since `internal/state` reads `channel_id` as an unknown-safe key; the relay must drop any validation that requires it.
- D11 no further modules: not verified, no fetch tool available; tier 0 with httptest: held, as the existing `api_test.go` shows.
- Constraints:
  - `PATTERN-daemon-wakeups`: the blocking flock is held; liveness is broken (3).
  - `PATTERN-tolerant-records`: the token file goes through `state.WriteJSON` (held).

**Tool calls**
- Bash over `internal/hubregistry/hubregistry.go` (`Record`, `Lookup`, `Live`), `credentials.go`'s state calls and JSON keys, `StopRelay` callers and the setup fields. My context lacked shortname uniqueness, the token-file shape and the stop callers.
- ToolSearch for WebFetch, to check `conversations.invite`'s user-token scopes. It was unavailable, so finding 13 stays unverified.
