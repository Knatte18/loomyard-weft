```yaml
lens: way-forward
outcome: reported
```

Lens: way-forward, covering every refusal, stop, reset and recovery step the cards add, and whether its named next step can succeed.

**Findings**

1. **MEDIUM** — Card 9, `RunRelay` step 6 (re-exec): a failed exec leaves the relay polling with no lock, and no card names a recovery.
   - Step 6 releases the lock in `beforeExec` and gives no step for an exec that then fails. The old image keeps polling without the lock.
   - A later `EnsureRelay` then sees `RelayLive` false, because the lock reads free, and starts a second relay. Two relays poll and each message is forwarded twice, which breaks D3's "only one relay polls".
   - `selfexec`'s `Watcher.CheckWith(beforeExec, failed)` has a `failed` callback for exactly this case. The card's seam is `CheckBinary func(beforeExec func()) error`, so it drops that callback, and no step retakes the lock or ends the relay.
   - Evidence: card 9 RelayDeps and step 6; `internal/selfexec/selfexec.go` `Watcher.CheckWith` (from the earlier spawn lookup).

2. **MEDIUM** — Card 12 with card 9: a relay that dies at once is started again on every prime dobby tick, with no cap on attempts.
   - While presence is `slack`, `EnsureRelayWhileAway` spawns again whenever the lock is free. A relay that exits early is re-spawned each tick: `AwaitReparent` times out, `RunRelay` errors before the first read, or `ErrRelayInStrandPane`.
   - Each restart spawns two processes and logs at Info.
   - `PATTERN-child-processes` says "A retry loop around a real spawn caps attempt COUNT". `PATTERN-daemon-wakeups` says a wake that spawns doubles its wait while nothing changes.
   - The plan sets no cap or backoff, and names nothing the operator is told after repeated failures.
   - Evidence: card 12 Intent, card 9 `EnsureRelayWhileAway`, `pattern/PATTERN-child-processes.md:16`.

3. **MEDIUM** — Card 7 and card 6: an inbound Slack refusal such as `invalid_auth`, `token_revoked`, `channel_not_found` or `not_in_channel` from `History` or `Latest` has no defined end.
   - Card 6 says the relay "counts only `*APIError` toward D5's cap of 5". That cap is outbound only (card 8).
   - Card 7 does not say how `inbound.poll` treats an `*APIError`. Read with card 9 step 5, it is retried forever with backoff.
   - Nothing reaches the orch or the operator naming `lym slack login` or `lym slack setup`. The only trace is `LogPath` and a stale `LastSlackOK` in `status`.
   - A refused token or a kicked bot is a state the operator must act on. The relay keeps polling and never names the next step.
   - Evidence: card 6 failure-kinds bullet; card 7 step 3; card 9 step 5.

4. **LOW** — Card 10, `setup`: only the not-a-member case of `ConversationInfo` names a way forward.
   - For a private channel the bot has not been invited to, Slack answers `channel_not_found`, not `is_member: false`. A missing `groups:read` scope answers `missing_scope`.
   - The card refuses only a non-member channel "naming `/invite @<app>`". It gives no way forward for `channel_not_found`, `missing_scope`, or a `UserInfo` `user_not_found`.
   - Evidence: card 10 `setup` bullet.

5. **LOW** — Card 10, `login`: a token Slack rejects is refused "naming Slack's error code", with no next step. It could name the bot token (`xoxb-…`) from the app's OAuth page and a re-run of `lym slack login`. `PATTERN-refusal-way-forward`'s style asks for a trailing way-forward clause.
   - Evidence: card 10 `login` bullet.

6. **LOW** — Card 10, `away`: the refusal after `AwaitFirstRead` names the log or creel's refusal, but no next step.
   - D6 gives the pane check's next step ("wait until it is known … then run the verb again"). The away refusal names none.
   - The relay stays live and holds its lock, so `lym slack away` run again succeeds once creel accepts. The refusal should say so.
   - Evidence: card 10 `away` bullet; card 9 `AwaitFirstRead`.

7. **LOW** — Card 9: `ErrRelayInStrandPane` wraps creel's message and names no way forward, such as "start the relay with `lym slack start` from the operator's terminal or an orch's pane".
   - Evidence: card 9 step 4.

8. **LOW** — Card 9 with card 10: `AwaitFirstRead` "ignores a state whose pid is gone". When the relay ended after creel refused its first read, it waits out the full 30 s and refuses naming only the log.
   - Card 10 says `away` names "creel's refusal when the relay recorded one", but a dead relay's recorded `FirstReadRefusal` is ignored.
   - Evidence: card 9 `AwaitFirstRead`; card 10 `away`.

9. **LOW** — Card 10, `stop`: when `StopRelay` returns an error after presence is already `terminal`, no next step is named.
   - Running `stop` again would progress, because `SetPresence` is unchanged and sends no notices before `StopRelay` runs again. The card does not say so.
   - Evidence: card 10 `stop` bullet.

10. **LOW** — Card 5: `LoadConfig` refuses `remind_after_min` below 1 "naming the key and the floor", with no way forward, such as setting it to 1 or more with `lym config slack`.
    - Evidence: card 5, second bullet.

**Way forward per refusal, and whether it progresses**

| Card | Refusal | Next step | Progresses |
|---|---|---|---|
| 4 | `RequireConfigured`, no token | `lym slack login` | yes |
| 4 | `RequireConfigured`, no setup | `lym slack setup --channel --operator` | yes |
| 7 | 4 KiB body | "send a shorter message" | yes |
| 7 | unknown slug | beam list | yes |
| 7 | 5-minute undelivered | "send it again" | yes, once creel accepts |
| 8 | 5× `APIError` | line naming `lym creel log --to` | yes |
| 8 | 24 h skip | line naming `lym creel log --to` | yes |
| 10 | pane, unknown | "wait until known, run again" | yes |
| 10 | pane, non-orch or orch | operator's terminal or orch pane | yes |
| 10 | `login`, `setup`, `away` | — | see findings 4–6 |
| 11 | bring-up warning | `lym slack start` | yes |
| 13 | anchor `fail` paths | `lym slack start` joined | yes |
| 13 | anchor change with relay live | — | not blocked: the prime liveness checks and `dobbyPassInFlight` do not count the relay, which the double fork reparented away from the dobby |

**Tool calls**
- grep of `internal/fabriccli/anchor.go` and `anchordeps.go` for liveness: to check whether a live relay blocks the prime anchor change. Not in context.
- `internal/fabriccli/anchordeps.go:426-470`: `dobbyPassInFlight`, to check whether the relay counts as a pass in flight. Not in context.
