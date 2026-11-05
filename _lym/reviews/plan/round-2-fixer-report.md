# Round 2 fixer report — plan

## Changes

- **F1 (BLOCKING), fixed differently in two respects.** Card 09 `_lym/plan/09-slackengine-relay.md`:
  - The cap bullet now says that `starter.ensure` writes `LastSpawnAt` and `NextStartAt` in one `RelayState` read-modify-write before it calls `spawn`, and returns the spawn's error after that write. A spawn that fails to start therefore counts as one failure at the next due start.
  - The "It then spawns … and sets `NextStartAt`" sentence now gives that same order.
  - Two `TestEnsureRelay` rows were added: a failing spawn, and a reset after an accepted first read.
  - *Row count differs from the proposal.* The proposal asked for "five `whileAway` starts … call it exactly five times and the fifth answers gave-up". By the card's own rule ("It then spawns, unless the count reached five"), the fifth failure is judged at the sixth start. The row is therefore six starts, five spawn calls each returning the spawn's error, and a sixth start that answers `gave-up` without a call. The existing dies-before-first-read row read "five `whileAway` starts … ends in `gave-up`" and had the same off-by-one, so it was corrected to "the sixth start answer[s] `gave-up`".
  - *Reset placed differently from the proposal (self-check, fix-proposal lens).* The proposal gave the reset to the starter, which sets `StartFailures` to 0 when a due start finds `FirstReadAt` later than `LastSpawnAt`. That leaves a gap. A healthy relay's re-exec writes `FirstReadAt` zero at step 3, and if the new image dies, a stale count of four reaches `gave-up` after two failures days apart. The reset is now in the relay's step 4 instead: the accepted-first-read write sets `StartFailures` to 0 in the same read-modify-write. The cap bullet names step 4 as the reset. A due start that finds a later `FirstReadAt` counts no failure. `EnsureRelay` without `whileAway` still resets the cap before it spawns.
  - A `TestRunRelay` row covers the step-4 reset. In the `TestEnsureRelay` reset row, the fake now writes `StartFailures` 0 as step 4 does.
- **F2 (MEDIUM), fixed as proposed.** Card 03 `_lym/plan/03-creel-seams.md`:
  - `internal/creelcli/doc.go` moved from **Extends:** to **Edit:**, the empty **Extends:** group was dropped, and `internal/creelcli#goCallTimeout` was added to **Edit:**.
  - The Intent now rewords the doc's "Every call passes a 30-second timeout …" sentence and `goCallTimeout`'s "bounds every request a GoClient makes" comment, so both exempt `ReadWait`.
  - The ImpactSummary names both rewordings.
- **F4 (LOW), fixed as proposed.** Card 10 `_lym/plan/10-slack-cli.md`: the `clitree_test.go` sentence now says that `newRoot`'s `Long` "Available modules" list gains `slack` beside `creel`.

F3 is `decision`-class and is left out of the dispositions above. The parent's recorded decision ("Take F3's fix as written", decision record, Added after Discussion, 2026-11-05) was applied to card 07 `_lym/plan/07-slackengine-inbound.md`:
- A `Hub.Paused` error after an accepted post is logged at Warn once per episode, sends no paused answer and moves the cursor.
- A `Hub.Beams` error comes before any post, so it leaves the cursor before the message and is retried with the backoff.
- One `TestInboundPoll` row was added.

## Self-check

- fix-consistency, LOW, card 09: "It then spawns … sets `NextStartAt`" contradicted the new write-before-spawn order. Fixed as proposed: reworded to write, then spawn.
- fix-consistency, LOW, card 12 line 25: "five relays in a row that never made their first read" missed the failing-spawn path. Fixed as proposed: now "five spawns in a row with no accepted first read, a spawn that fails to start included". The fix-affected-set LOW raised the same point and was merged here.
- fix-affected-set, LOW, card 12 line 25: duplicate of the line above. Fixed there.
- fix-claims: no findings.
- fix-proposal, MEDIUM, card 09: a re-exec zeroes `FirstReadAt`, so the starter-only reset lets failures that are not in a row reach `gave-up`. Fixed with my own change: the reset moved to the relay's step 4 write (see F1). I did not take the lens's other remedy, keeping `FirstReadAt` across a re-exec. A re-exec that dies before its first read must still count as a failure, and that remedy would hide it.
- fix-proposal, MEDIUM, card 07: a lasting `Hub.Beams` error holds the cursor with no bound, no Warn log and no Slack answer. Deferred, see below.
- fix-proposal, LOW, card 07: "read only for a text opening with an at-sign slug" did not state when the poll reads `Hub.Beams`. Fixed as proposed: the sentence now says the poll calls `Hub.Beams` only for a text opening `@<slug> `.

Every fork returned a report.

## Disputed

None.

## Deferred

- **Self-check, fix-proposal MEDIUM, card 07, the `Hub.Beams` error path.** The case holds. `Hub.Beams` comes from `hubgeom.DobbyStart(prime).Beams`, which can fail on every read. Holding the cursor before an `@slug` message then blocks every later message, `away`, `back` and `status` included, with no bound and no word to the operator. The lens's remedy is to give it the refused post's path: Warn once per episode, then after 5 minutes a Slack answer and the cursor moved past the message. That changes the text of the parent's recorded F3 decision, which says the poll "returns it with the cursor left before the message and retries with the backoff". Only the parent can amend it.
