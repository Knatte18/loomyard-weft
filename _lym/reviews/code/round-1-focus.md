---
round: 1
focus:
  - "internal/slackengine/inbound.go `handle`: the write `in.awayFrom[orch] = true` — is it made under `in.mu`, as `takeAwayFrom` reads and swaps the map under it, and is `handle` reached from a goroutine other than the wake loop's?"
  - "internal/slackengine/relay.go `noticePresenceChange`: `takeAwayFrom` runs before the early returns, so the set is cleared whether or not a notice follows; settle whether an orch skipped for a notice of slack can lose a notice it should still get (a presence change back to terminal, a failed notice, a beam orch not yet in `beams`)."
  - "internal/slackengine/backoff.go `afterEmptyPoll` and outbound.go `read`: settle whether `elapsed >= floor` is a sound test that the poll waited out its time given `outboundReadTimeout`, a floor raised by a rate limit, and the `errors.Is(err, ErrWaitTimeout) && !handlerRan` branch, and that no path can run the loop hot."
  - "internal/shuttleengine/claudeengine/settings.go `creelStopHookCommand`, `creelOpenStopHookCommand`, `creelStopDelivery`: settle whether the turn hook runs on every Stop before the creel hand-over, `in` is read once, the exit status and output contract in the doc comments still hold for each of the three command shapes, and an empty `turnHookCommand` leaves the old behavior."
  - "Scope against the board entry's Idea: the diff should carry all three ideas (forward on every Stop, the late relay read, the presence notice that need not wake the orch); settle whether any is missing or whether the diff changes something the entry does not license."
  - "Tests: settle whether each added behavior (turn hook before hand-over, `afterEmptyPoll`, the skipped presence notice) has a test, and whether any new test duplicates an existing one."
  - "Doc comments in doc.go files and the changed functions: settle whether they match the code as the diff leaves it (the merged Stop command's order, the backoff description)."
---

The decision record holds only the board entry under `## Goal`, so the entry is the measuring stick.
