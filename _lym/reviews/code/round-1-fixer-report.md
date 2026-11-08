# Fixer-report, round 1

## Changes

- F1: `relay_test.go` gets a table of two rows.
  One pins the `presence != PresenceSlack` reset: a message turns the presence and the presence goes back to terminal in the same wake, and every orch gets the terminal notice.
  The other pins that a wake which sends no notice still forgets the kept orchs, so a later slack notice reaches them.
  The `err == nil` condition is pinned in `inbound_test.go` through F2 and F3 (a failed post leaves the kept set empty).
- F2: `forward` returns `(handled, posted, err)`, and `handle` marks the orch in `awayFrom` only when a post stood.
  `inbound_test.go` has a table over `TakeAwayFrom` (exported in `export_test.go`) with rows for an oversize message, an empty message and a presence already slack.
- F3: `inbound` gets an `awayPending` set under `in.mu`.
  `handle` records the orch there when it turns the presence and moves it to `awayFrom` when a post stands.
  It drops the entry when the message finishes unposted.
  Rows: the first post fails and the retry stands, and a message given up on does not mark a later one.
- F4: `spec.go` `TurnHookCommand` doc reworded as proposed.
- F5: `outbound_test.go` gets a row for the reset after a long poll (waits 2x, 4x, none, 2x) and a row for a floor raised by a rate limit with a poll of exactly the floor and one nanosecond under it.
- Self-check fix: the `forward` doc line over 120 characters is split.
- Gate repair: the split left a sentence of the `forward` doc wrapped over two lines, so the doc now states the three handled-and-not-posted cases as three one-line sentences.

`lym gate test ./internal/slackengine ./internal/shuttleengine` (tier0) passed before the self-check; the two doc-line changes touch a comment only.

## Disputed

- Self-check `fix-code-correctness` finding 1 and `fix-proposal` finding 1 (stale `awayPending` entry), ground `premise false`:
  the later message that posts to the orch does wake it, so skipping that orch's next slack notice is right for that post.
  When the presence went back to terminal in between, the next message turns it again and `turned` stands anyway (`inbound.go` `handle`, `awayOnOperatorMessage`).
- Self-check `fix-code-correctness` possible flake and `fix-proposal` finding 2 (`beamReads` sync), ground `premise false`:
  the outbound loop is parked at a wait of at least `Floor` on the manual clock, and `driveUntil` advances only `WakeFloor` (`relay_test.go` `awaitOutboundWait` comment), so only wake-loop wakes call `Beams` in that row.

## Deferred

None.

## Rulings

None.

## Self-check

- fix-code-correctness, LOW, stale `awayPending` entry: disputed, premise false.
- fix-code-correctness, LOW, over-long `forward` doc line: fixed as proposed.
- fix-code-correctness, possible flake on `beamReads`: disputed, premise false.
- fix-consistency: no findings.
- fix-proposal, LOW, stale `awayPending` entry: disputed, premise false (merged with the code-correctness entry).
- fix-proposal, LOW, `beamReads` sync: disputed, premise false (merged).
- fix-proposal, LOW, the F5 row discards the result of the read after the rate limit: fixed differently, no change needed: the row's later assertions fail if the floor was not raised, so the read's result is pinned by them.
- fix-affected-set: no findings.
