# Lens review — Plan-Write, slack-push-channels

Every lens fork returned a report; each is saved beside this file as `lens-review.lens-<lens>.md`.
Duplicates across lenses are merged on one line, which names every lens that raised the finding.

- coverage 1, MEDIUM: no row for a hint file holding text — fixed as proposed: card 10 gains a row whose `C2` hint holds `deploy prod`, and only Slack's text, or nothing, is forwarded.
- coverage 2, MEDIUM: no row drains a `resync` hint, retries a failed resync read or joins on a resync `not_in_channel` — fixed as proposed: card 10 gains one row covering all three.
- coverage 3, MEDIUM: no row asserts the hint-write failure notice — fixed as proposed: card 8's failing-writer row asserts one notice naming B's inbox and the error across two failures.
- coverage 4, MEDIUM: no row fails the post-connect resync write — fixed as proposed: card 8 row, paid at the next ping.
- coverage 5, MEDIUM: `link_disabled` notice unreached — fixed as proposed: card 8's notice row adds a `disconnect` with reason `link_disabled`.
- coverage 6, MEDIUM: pong timeout on wall time and unreached — fixed as proposed: card 8 races `Ping` against the clock's `After(pongTimeout)`, and a row's `Ping` blocks past it.
- coverage 7, MEDIUM: no row checks the outbound wait backs off while owed reads or invites fail — fixed as proposed: card 12 row asserts pending waits 10, 20, 40, 60, 60 for both.
- coverage 8, MEDIUM: no-setup half of the missing-config exit dropped — fixed as proposed: card 12's reworked row adds a relay with both tokens and no `setup.json`.
- coverage 9 with row-validity 8, MEDIUM: failed live-hub read after login's write and an identical repeat login have no row — fixed as proposed: card 5 rows for both.
- coverage 10 with row-validity 8, LOW: refused `apps.connections.open`, failed `bots.info` and dial transport failure unreached — fixed as proposed: card 5 rows for each.
- coverage 11, LOW: refused and transient ensure in the outbound unreached — fixed as proposed: card 11 rows for both.
- coverage 12, LOW: `Ensure` on a held entry with another operator unreached — fixed as proposed: card 9 row.
- coverage 13, LOW: `botUser` failure, a list that no longer finds the name and a rename refused with another code unreached — fixed as proposed: card 9 rows for each, the last with `not_authed`.
- coverage 14, LOW: drain's missing-setup wording unstated — fixed as proposed: card 10 states it names `lym slack setup --operator <user-id>`, with a row.
- coverage 15, LOW: `SlackGeometry` hostname failure and `status` map error unreached — status half fixed as proposed: card 6 row over a map holding `{`.
  The hostname half is disputed, every fix worse: `machinename.Name` reads `os.Hostname` (`internal/machinename/machinename.go:12`), so reaching it needs a hostname seam, a package-level var `PATTERN-test-speed` "Time is injectable" bars or a new `SlackGeometry` parameter on every caller; the branch only propagates that function's error, which already names its way forward, and card 4 now says so.
- coverage 16, LOW: ctx ending during the socket-lock wait unreached — fixed as proposed: card 8 row.
- coverage 17 with forced-edits 10, LOW: `wakeLoop`'s file unnamed — fixed as proposed: card 12 pins `wakeLoop`, `relayStart`, `missingTokenExit` and `holderRun` to `relay.go`, which stays scanned.
- way-forward 1, MEDIUM: a beam with no open channel blocks the whole batch — fixed as proposed: card 11's `postMessages` skips that message unhandled, posts the rest and releases the read at the batch's end, with a row.
- way-forward 2 with guard-reach 4, MEDIUM: the dropped-message line has no channel after a refused ensure — fixed as proposed: card 11 posts it to the prime's channel, or mails the hub's orch a `[slack relay]` notice when that is refused too, with a row.
- way-forward 3, MEDIUM: `missingTokenExit` leaves the relay down — fixed as proposed: the line adds `then lym slack start`, asserted in card 12's row.
- way-forward 4, MEDIUM: a re-run login cannot repeat the relay ends — fixed as proposed: card 5's error names `lym slack stop` then `lym slack start` in each hub.
- way-forward 5, LOW: the refusal cap drops a message before a rename in Slack can land — disputed, every fix worse: D8 fixes the count, "Each refused post counts once toward the existing per-message cap of five Slack refusals" (`_lym/discussion/decision-record.md:362`), so exempting the ensure or slowing the count departs from the record; the dropped line names `lym creel log`, so the mail stays readable.
- way-forward 6, LOW: a retire with a refused archive renames again from the D4 name — fixed as proposed: card 9 records the dated name once the rename stands and skips the rename on retry, with the row asserting no second rename.
- way-forward 7, LOW: the other-orch refusal names no way forward — fixed as proposed: card 9's notice names renaming that channel in Slack.
- way-forward 8, LOW: a hub's geometry failure reads as a broken hubs record — fixed as proposed: card 5's `liveHubGeometries` names the failing hub and its geometry error, the hubs record only for a failed `Live` read.
- way-forward 9, LOW: the partial-token refusal leads into a second refusal — fixed as proposed: card 5's `RequireConfigured` says login takes both tokens in one run.
- way-forward 10, LOW: a transient prime ensure failure at start is never retried — fixed as proposed: card 12 remembers it for the outbound wakes, with a row.
- way-forward 11, LOW: one refusal notice mark shared across channels — fixed as proposed: card 10 keeps the mark per orch, with a row.
- way-forward 12, LOW: an undecodable map gets no way forward in `status` — fixed as proposed: card 4's decode error names moving the file aside, asserted in cards 4 and 6.
- row-validity 1 with guard-reach 2 and 3, BLOCKING: card 7's stamp rows cannot reach `afterRead`, and the idle row cannot fail — fixed differently: the proposed `newRelayRun` Edit was not made, since card 12 rewrites that helper and both rows need only `blockReadWait` cleared; card 7's rows clear it, stop from a hook once their condition holds, and the idle row seeds presence `slack`, `LastSpawnAt` and `NextStartAt` and judges through `NewStarter` over `fakeSpawn`.
- row-validity 2, MEDIUM: the at-once `fakeClock` makes multi-loop wait rows non-deterministic — fixed as proposed: card 8 adds `manualClock`, whose waits deliver only when a row advances the time, used by cards 8 and 12, and `fakeSocket` and `fakeDialer` expose ack and dial channels to order a row's steps.
- row-validity 3, MEDIUM: the five-refusals row's `callCount() == 6` breaks under the keeper — fixed as proposed: card 11's harness entries record `Operator` `U1`, and the row counts `chat.postMessage` calls through `logged`.
- row-validity 4, MEDIUM: no fake method seeds foreign or archived channels — fixed as proposed: card 7 adds `fakeSlack.seedChannel`, used by card 9.
- row-validity 5, MEDIUM: no per-method hook for the invite-order row — fixed as proposed: card 7 adds `fakeSlack.onMethod`, used by card 9's row.
- row-validity 6 with forced-edits 9, MEDIUM: `LiveHubs` has no nil meaning and `newRelayRun` misses it — fixed as proposed: cards 9 and 10 state a nil `LiveHubs` lists the own hub alone, card 10 lists `newRelayRun`, and card 11's keeper gets an explicit `LiveHubs`.
- row-validity 7, MEDIUM: login rows lack a slackcli fake socket and cannot see `bots.info`'s form — fixed as proposed: card 5 adds `fakeLoginSocket`, a dial counter, form recording in `slackRequest`, a `bot_id` in the `/auth.test` default and an assertion that `/bots.info` got `bot=B1`.
- row-validity 9, LOW: two card 11 rows repeat existing ones — fixed as proposed: folded into `MessageSlackRefusesFiveTimesIsDroppedWhetherOrNotTheReplacementLinePosts` and `MailOlderThanADayIsAcknowledgedUnpostedWithOneCountLine` as channel assertions.
- row-validity 10, LOW: the stop row never runs `StopRelay` — disputed, every fix worse: `StopRelay(geom, callerPID)` holds no Slack client (`internal/slackengine/start.go:143-145`), so no archive can come from it, and running it ends real processes through `runproc.EndRole`, which a `tier0` row may not do (`PATTERN-test-speed` "`tier0` tests spawn nothing").
- row-validity 11, LOW: the re-exec lock row has no synchronisation — fixed as proposed: card 12's row waits on `fakeDialer`'s next dial, which follows the holder retaking the lock.
- row-validity 12, LOW: card 1 left the documented answers to be fetched — fixed as proposed: card 1 records each answer's JSON.
- row-validity 13, LOW: a stale row name — fixed as proposed: card 6 renames it `PrintsPresenceRelayPauseOperatorChannelsAndLastHealthy`.
- guard-reach 1, MEDIUM: the envelope limit cannot fire on a stall — fixed as proposed: card 8 runs `route` off the serving loop under the clock's `After(envelopeLimit)`, owing the found hub or every hub of the last good read, with a blocking-writer row.
- guard-reach 5, MEDIUM: the outbound wake could run beside the drain — fixed as proposed: card 12's `afterRead` only signals `wakeLoop`, whose goroutine does the work, with a row.
- guard-reach 6, LOW: one keeper instance not stated — fixed as proposed: card 10 and the overview's Shared Decisions state one keeper shared by the inbound, the outbound and the wake loop.
- guard-reach 7, LOW: a half-open connection before `hello` hangs the dial — fixed as proposed: card 2's `NewDialer` takes a `helloTimeout`, production passing `HelloTimeout`, with a row.
- forced-edits 1, MEDIUM: re-sign arrows missing on card 11's outbound methods — fixed as proposed: arrows on `post`, `postMessages`, `refused` and `reportStale`, dropping `setup` and giving `refused` the channel.
- forced-edits 2, LOW: `API`'s bot-token-only comments — fixed as proposed: card 1 lists `API` and `NewAPI` and rewords them and the file header.
- forced-edits 3, LOW: `TokenFileName` and `SetupFileName` comments — fixed as proposed: card 5 lists the first, card 6 the second.
- forced-edits 4, LOW: `backoff.go`'s two-loop comments — fixed as proposed: card 12 lists `sharedFloor` and rewords the file header.
- forced-edits 5, LOW: `starter.ensure`'s comment — fixed as proposed: card 7 lists it.
- forced-edits 6, LOW: file headers of rewritten files — fixed as proposed: cards 5, 10, 11 and 12 each reword their file's header.
- forced-edits 7, LOW: `template.yaml`'s comment — fixed as proposed: card 6 lists the file and rewords the comment.
- forced-edits 8, LOW: `hubgeom/doc.go` under `Uses` — fixed as proposed: card 4 moves it to `Extends` and adds the machine name and socket lock.
- forced-edits outside the lens, LOW: the hint's temp file departs from `PATTERN-lym-files` — fixed as proposed: the overview's Shared Decisions record the departure and card 8 cites it.
