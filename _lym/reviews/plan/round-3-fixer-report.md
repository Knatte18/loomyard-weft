# Fixer report — plan, round 3

## Changes

- **F1 (BLOCKING), fixed as proposed.**
  In `09-slackengine-relay.md`, the starter now judges each spawned relay by its lifetime, `LastSlackOK` minus `LastSpawnAt`, against the named constant `relayHealthyLifetime` (10 minutes) in `start.go`.
  A zero `LastSlackOK`, or one not after `LastSpawnAt`, reads as zero.
  A lifetime under the constant counts one failure; a lifetime at or over it sets `StartFailures` to 0.
  Step 4 still writes `FirstReadAt` but no longer resets `StartFailures`.
  The sentence that put the reset in the first read is replaced by the lifetime rule, and the spawn-before-write sentence now speaks of `LastSlackOK`.
  - Rows changed:
    - The `TestRunRelay` row now asserts that `StartFailures` 4 is kept.
    - The `TestEnsureRelay` reset row is replaced by an eleven-minute `LastSlackOK` row that leaves `StartFailures` 0, plus a ten-minute boundary case.
    - A new row: a relay that dies one minute after its first read, over five due starts, makes the sixth answer `gave-up`.
    - The "dies before step 3" row now uses an earlier relay's `LastSlackOK`.
  - Twin fixed: `12-dobby-relay-tick.md`'s `gave-up` sentence now says "five spawns in a row whose relays each lived under ten minutes".
- **F2 (MEDIUM), fixed as proposed.**
  In `07-slackengine-inbound.md`:
  - The `Hub.Beams` sentence now names both call sites, the routing call and `SetPresence`'s `beams` argument. The pre-post rule stays on the routing call.
  - The presence bullet gives each `SetPresence` result an outcome:
    - `changed` true with an error: logged at Warn once per episode, answered, and the cursor moves.
    - `changed` false with an error: the poll returns it with the cursor left.
    - `status` does the same when `ReadPresence` errors.
  - The `SetPresence` paragraph states both error shapes.
  - Rows added:
    - `TestInboundPoll`: an `away` with a `Hub.Beams` error after the write, an `away` with a failed write, and a `status` over an unknown stored value.
    - `TestSetPresence`: a `beams` error after the write.
- **F3 (LOW), fixed as proposed.**
  `TestInboundPoll` gains the `@payments x` row with a `Hub.Beams` error, and the poll after recovery forwards it once.
- **F4 (LOW), fixed as proposed.**
  The 30-minute clamp is gone. The wait is "doubled for each counted failure, so 1, 2, 4, 8 or 16 minutes before the fifth failure ends in `gave-up`". F1 leaves the count's reach unchanged.
- **F5 (LOW), fixed as proposed, after F1.**
  - A `whileAway` start before `NextStartAt` answers `waiting`. Card 12 needs no change, since it handles only `gave-up`.
  - The lifetime rule's boundary is stated: at or over the constant counts as healthy, and a `LastSlackOK` not after `LastSpawnAt` reads as zero.
  - In the "down relay answers `started`" row, the fake spawn now finds `LastSpawnAt` and `NextStartAt` already stored when it is called.
  - The `TestEnsureRelay` lead now says each fake relay moves the fake clock forward before it writes `FirstReadAt` or `LastSlackOK`.
  - The wait row asserts `waiting`.
- **F6 (LOW), fixed as proposed.**
  The `ReadWait` bullet in `03-creel-seams.md` now says:
  - It returns nil once the acknowledgement of the read it last made is confirmed.
  - A lost connection sends the read again, through `CallWatching`, up to `MaxReconnects`, so the handler can run twice. Card 8's outbox record keeps that from posting twice.
  - A refused confirmation, the told timeout and exhausted reconnects return their error.

  The `creelcli/doc.go` sentence carries the same contract.
- **F7 (LOW), fixed as proposed.**
  `10-slack-cli.md` states the two Use strings:
  - `setup --channel <id> --operator <user-id>`, with both flags required.
  - `relay --hub-path <hub> --prime <name>`, with both flags required, and `--fork` and `--after` as optional flags the Use string does not list.

## Self-check

- fix-consistency — LOW — fixed as proposed: `04-slackengine-files.md`'s `RequireConfigured` way forward now writes `<user-id>`, matching card 10's Use string.
- fix-proposal — MEDIUM — fixed with my own change.
  The lens proposed answering a lasting presence error at once and moving the cursor. I did not make that change: it would also drop an `away` whose write failed for a passing reason, against F2's retry outcome.
  Instead, a presence error that leaves the cursor gets the 5-minute bound that card 07 already applies to a failed post: once the failure is 5 minutes old, the relay answers quoting the error and moves the cursor past the message.
  The `status` unknown-value row now covers the 5-minute pass.
- fix-proposal — LOW — fixed with my own change.
  The lens proposed either stating an accepted bound or capping total spawns per episode. I stated the bound in card 09 and did not cap total spawns, since a total cap would stop restarts of relays that ran as relays.
  The count caps the loop of quick failures. Restarts of relays that each lived `relayHealthyLifetime` are bounded in rate, at most one per `relayHealthyLifetime`, while presence stays `slack`.
- fix-claims — no findings.
- fix-affected-set — no findings; it left F3, F4 and F5 unchecked, since none rests on a set.

## Disputed

None.

## Deferred

None.
