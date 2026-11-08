```yaml
lens: coverage
outcome: reported
```

Coverage lens findings for the slack-push-channels plan. Defects only; `DR` is `_lym/discussion/decision-record.md`.

1. MEDIUM — D1 hint without text. **No row writes a hint file that holds text and checks that only what Slack returns is forwarded.** The criterion is DR:531 ("a hint file holding text is never forwarded as text"). Card 10's hint rows (`TestInboundRead`) use only empty hints via `harness.hint`. Fix: add a row with a `C2` hint whose content is operator-like text, asserting that the forwarded body is the Slack message's text and not the file's.

2. MEDIUM — D1 resync hint, D2 resync read. **No card 10 row drains a `resync` hint.** The criterion is DR:547 ("a resync hint after a reconnect, forward the top-level messages after each cursor … replies"). Card 10 says "`resync` owes every open channel a read with its 24-hour thread scan", but every resync row reads through `Start`. Card 8 only asserts that the hint file is written. A resync hint that is dropped, or read without its scan, passes every row. The failed-resync-read retry (DR:545) and the resync `not_in_channel` join (DR:605) have no row either.

3. MEDIUM — D1 hint-write failure notice. **No row asserts the notice for a failed hint write.** The rule is DR:64 ("one `[slack relay]` notice per episode naming the inbox directory and the write's error"). Card 8's failing-hint-writer row (`TestSocketHolder`) asserts the ack and the owed resync only.

4. MEDIUM — D1 failed resync write after a connection opens. **No row covers it.** The rule is DR:60 ("A resync hint that fails after a connection opens is owed the same way"). Card 8's rows cover a failed event-hint write and "each new connection writes a resync hint". None makes that post-connect resync write fail and checks it is paid at the next ping.

5. MEDIUM — D1 `link_disabled` disconnect. **The disabled-connection notice branch has no row.** Card 8 Intent carries the branch ("a `disconnect` whose reason is `link_disabled` … mails … one notice … naming `lym slack login`"). Its only notice row is a dial answering an `*APIError`. The criterion is DR:535 ("refuses or disables").

6. MEDIUM — D1 pong timeout, Constraints test-speed. **The 10-second pong timeout runs on wall time, not the injected clock, and no row reaches it.** Card 8 says "calls `Ping` under a `pongTimeout` context", which is a real-time `context.WithTimeout`. Its row only scripts a ping error. The criterion is DR:533 ("a pong missing 10 seconds after a ping opens a new connection"). A broken timeout passes, and a test that tried to reach it would wait out a production duration (DR:484, `PATTERN-test-speed` "Time is injectable"). Fix: derive the deadline from the clock seam, for example a `pongTimeout` wait on `Clock.After` raced against `Ping`, and add a row whose `fakeSocket.Ping` blocks.

7. MEDIUM — D2 failed owed read and the outbound wait. **No row checks that the wait backs off while owed reads keep failing.** The rule is DR:137 ("A failed owed read counts as finding nothing for the outbound read's wait … back off to sixty seconds"). Card 12 states it in prose ("None of that work shortens the outbound read's wait"), but no `TestRunRelay` row records the outbound waits while an owed read fails. A failed invite counting as nothing (DR:271) is likewise unasserted.

8. MEDIUM — D3 relay with no setup file. **The no-setup half of the old row is dropped.** Card 12 rewrites `HubWithNoTokenOrNoSetupIsRefusedAndTakesNoLock` to cover only a relay with a bot token alone. `missingTokenExit`'s branch naming `lym slack setup --operator &lt;user-id&gt;` has no row, though `RequireConfigured` now runs after the lock and the first read.

9. MEDIUM — D3 login after the token write. **The failed live-hub read has no row.** Card 5 carries "A failed live-hub read after the write answers an error naming the hubs record and that way forward; the tokens stay written". Its rows cover a replacing login and a first login only. A login that repeats the identical stored tokens, which should call no `stop`, also has no row.

10. LOW — D3 confirmations. **Several login checks have no row.** These are a refused `apps.connections.open` going through `tokenRefusal` "naming which kind Slack rejected", a failed `bots.info`, and a dial transport failure. Card 5's only refusal row is a refused `auth.test`.

11. LOW — D5 refused beam ensure, card 11 branches. **No `TestOutboundRead` row reaches either ensure failure.** One is a refused ensure (`errChannelRefused`) counted as a post refusal toward the cap. The other is a transient ensure failure that releases the read. The rule is DR:293 ("a refused beam's channel at its next routed mail").

12. LOW — D5 invite on a held channel. **No row runs `Ensure` on an open entry that records another operator or none.** Card 9 says ensure of a held channel invites the operator then, but only `InviteOperators` is exercised.

13. LOW — card 9 branches. **Three keeper branches are unreached.** These are `botUser`'s `auth.test` failure, "A name `conversations.list` no longer finds is an error", and a rename refused with a code other than `name_taken`, which DR:444 says is retried like an archive. Only a refused archive has a row.

14. LOW — Constraints, refusal way forward. **Card 10 does not state the drain's missing-setup error wording.** The constraint is DR:481 (reword every `lym slack setup --channel` way forward). Card 10 deletes `inbound.poll`, whose message named `--channel`, and reads `setup.json` "at each drain". It does not state that the new missing-setup error names `lym slack setup --operator &lt;user-id&gt;`, and no row asserts it.

15. LOW — card 4 and card 6 branches. **Two error branches have no row.** One is `SlackGeometry` refusing a machine whose name cannot be read. The other is `status` returning an error for an unreadable channel map.

16. LOW — card 8 lock wait. **The branch where ctx ends while the holder blocks on the socket lock has no row.** That goroutine should release the lock at once when it gets it.

17. LOW — card 12, `PATTERN-daemon-wakeups`. **No card names the file `wakeLoop` lives in.** Card 12 says only that "`relay.go` stays on `unboundedWaitFiles`". If the loop goes in a new file, `cmd/lym/daemonwakeups_test.go` does not scan it and no card lists the registration. Fix: pin `wakeLoop` to `relay.go`, or add the new file to the list on card 12.

Tool calls:
- grep of production `hubregistry.Live/Record/Lookup` callers, to check the overview's Shared Decision on the hubs record directory; my context had no `hubregistry` caller.
- grep of `registryDir` in `internal/creelcli`, to confirm creel passes `standalonestate.UserStateDir`; my context lacked creel's seam.
