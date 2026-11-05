```yaml
lens: guard-reach
outcome: reported
```

**Guard-reach: each guard and whether it can be reached**

- Card 3, coded refusal `ErrProgramClaimInStrandPane`: reachable on Linux, and the `TestCreelCLI` subtest reaches it through `fail` and `exchange`. Off Linux it can never fire (finding 6).
- Card 4, `RequireConfigured` missing token or setup: reachable, and rows exist.
- Card 4, `ReadPresence` unknown value: reachable, and a row exists.
- Card 4, `SlackGeometry` unresolvable credential directory: reachable, but no row reaches it (finding 8).
- Card 5, `remind_after_min` below 1: reachable, and a row exists.
- Card 6, `RateLimitError`, `APIError` and plain error kinds: reachable, and rows exist.
- Card 7 — every guard here is reachable and has a row:
  - skip of non-operator, bot, thread and subtype messages;
  - unknown `@slug`;
  - body over 4096 bytes;
  - refused post and the 5-minute answer;
  - paused receiver;
  - cursor reset on a missing or changed channel;
  - the `[slack relay]` notice on an `APIError`, once per episode.
- Card 8 — reachable, with rows:
  - the sender filter (non-orch strand, EXTERNAL);
  - the 24-hour skip;
  - the cap of 5 `APIError` refusals;
  - the 25-second release.
- Card 8, the `ErrWaitTimeout` "empty read" branch: also reached after a read that was handled (finding 3).
- Card 9, step 1 refusals: reachable, with rows.
- Card 9, step 2 lock contention and the re-exec wait: reachable, with rows.
- Card 9, step 4 strand-pane end: reachable on Linux by a run by hand only. Never reached by a relay that went through the double fork, and never reached off Linux (finding 6).
- Card 9, step 4 retry on other refusals: reachable (unknown pane, daemon down).
- Card 9, step 6 `failed` retake: reachable, but also trips on a probe (finding 5), and its wiring does not type-check (finding 4).
- Card 9, `AwaitReparent` bound: reachable.
- Card 9, `AwaitFirstRead` "gone relay with a recorded refusal": can't occur for the relay `away` just started; it is reached only from a stale state (finding 1).
- Card 9, `AwaitFirstRead` bound: reachable.
- Card 9, `whileAway` cap and `gave-up`: unreachable in production (finding 2).
- Card 10, pane check (strand pane, orch allowed or not, unknown pane, unreadable strands, `StrandName` off Linux): reachable, with rows.
- Card 10, `login` empty token: reachable, but no row (finding 7).
- Card 10, `setup` refusals (`channel_not_found`, non-member, `missing_scope`, `user_not_found`, no token): reachable, with rows.
- Card 10, `relay --after` with a reparent error: reachable, with a row.
- Card 11, relay seam error warning: reachable, with a row.
- Card 12, tick spawn under test (`ErrSpawnUnderTest`) and presence `terminal`: reachable, with rows.
- Card 13, `start-relay` after `EndDobbies` on every fail path, and none before it: reachable, with rows.
- Recurrence: the other sites that end a `HubOwner` process are `orchcli` (`EndRole` of `RoleDobby` only, `internal/orchcli/cli.go:267`) and `fabriccli`'s `EndSparing`, which card 13 covers. Nothing else found unguarded.

**Findings**

1. **MEDIUM — design — card 9 `AwaitFirstRead` (used by card 10 `away`).**
   - Summary: the "relay gone with a recorded `FirstReadRefusal`, refuse at once" branch can't be reached by the relay `away` just started, so only a stale state left by an earlier relay reaches it.
   - Why: a relay that went through the double fork is in no pane, so the strand-pane refusal can't hit it. Every other refusal keeps the relay alive and retrying.
   - Effect: a leftover `RelayState` from an earlier relay run by hand in a non-orch pane makes `lym slack away` refuse before the new relay writes its fresh state.
   - Fix: scope the await to a state whose `StartTime` is not before the await began.
   - Evidence: `_lym/plan/09-slackengine-relay.md:53,55,70`; `_lym/plan/10-slack-cli.md:62`.
2. **MEDIUM — design — card 9 `whileAway` restart cap.**
   - Summary: `gave-up` can't be reached in production.
   - Why: step 3 writes `RelayState` "afresh", which wipes `StartFailures` and `NextStartAt` on every spawn that gets that far. A relay that dies before step 3 leaves the previous relay's `FirstReadAt` in place, so no failure is counted.
   - The test row passes only because its fake spawn writes no state.
   - Fix: step 3 keeps the starter's two fields, and a failure is counted against the starter's own spawn time.
   - Evidence: `_lym/plan/09-slackengine-relay.md:53,66,76`.
3. **MEDIUM — consistency — card 8 `outbound.read`, against card 3 `ReadWait`.**
   - Summary: the `ErrWaitTimeout` "empty read, keep the record, double the idle wait" branch is also reached after a read whose handler posted messages.
   - Why: creel's single timer spans the wait for mail, the handler and the ack confirmation. When it fires during the 25-second handler, `awaitAckConfirmation` returns `ErrWaitTimeout`.
   - Fix: either `ReadWait` bounds only the wait for mail, or `read` treats `ErrWaitTimeout` after its handler ran as a lost confirmation and returns the floor wait.
   - Evidence: `internal/creelengine/client.go:144-146,274,284-287`; `_lym/plan/08-slackengine-outbound.md:32,35-36`; `_lym/plan/03-creel-seams.md:38`.
4. **MEDIUM — consistency — cards 9 and 10, `RelayDeps.CheckBinary`.**
   - Summary: the seam is declared `func(beforeExec, failed func()) error`, but the production value it is wired to has a different shape, so the wiring as written doesn't compile and the `failed` branch has no well-typed path.
   - The production value is `selfexec.(*Watcher).CheckWith(beforeExec func(), failed func(error))`, which returns nothing.
   - Fix: use dobby's shape, `func(beforeExec func(), failed func(error))`.
   - Evidence: `_lym/plan/09-slackengine-relay.md:49-50`; `_lym/plan/10-slack-cli.md:65`; `internal/selfexec/selfexec.go:80`; `internal/dobbyengine/loop.go:64`.
5. **LOW — design — card 9, step 6 `failed` path.**
   - Summary: the retake uses `TryAcquireWriteLock`, so a `RelayLive` probe briefly holding the lock makes the old image log "another holder" and exit with no second relay running.
   - Probes come from every `whileAway` tick and from `status`. Step 2 guards against probes with `ProbeGrace`; step 6 does not.
   - Fix: `AcquireWriteLockWithin(lock.ProbeGrace)`.
   - Evidence: `_lym/plan/09-slackengine-relay.md:52,59,63`.
6. **MEDIUM — decision — cards 9 and 10, a relay run by hand off Linux.**
   - Summary: off Linux, `resolveSender` returns through `resolveFromEnvironment` before any pane check, and a program client's claim carries no strand name. So `ErrProgramClaimInStrandPane` never occurs there.
   - Effect: `lym slack relay` run by hand from a non-orch strand's pane is accepted as `SLACK` and polls Slack.
   - The bound stated in card 9 step 4 and card 10 holds on Linux only, and neither card says so.
   - Fix: state the Linux-only bound, or have the relay verb refuse off Linux through card 10's `LYM_STRAND_NAME` pane check.
   - Evidence: `internal/creelengine/identity.go:52-54`; `internal/creelcli/goclient.go:28-31`; `_lym/plan/09-slackengine-relay.md:55`; `_lym/plan/10-slack-cli.md:45`.
7. **LOW — consistency — card 10, `login` empty-token refusal.**
   - Summary: no `TestSlackCLI` row reaches it.
   - Evidence: `_lym/plan/10-slack-cli.md:58,74`.
8. **LOW — consistency — card 4, `SlackGeometry` unresolvable credential directory.**
   - Summary: no `TestSlackGeometry` row reaches it, though clearing `HOME` and `XDG_CONFIG_HOME` with `t.Setenv` would.
   - Evidence: `_lym/plan/04-slackengine-files.md:41,57`.

**Tool calls**

- Grepped the `runproc` end sites, dobby's `CheckBinary` field and `selfexec.CheckWith`: to check for unguarded recurrences of the relay-ending site and the shape of the re-exec seam, which the manifest lacked.
- Read `internal/tendercli/deps.go:100-120` and `internal/orchcli/cli.go:255-272`: to see which owners those end sites end, one hop past the manifest.
