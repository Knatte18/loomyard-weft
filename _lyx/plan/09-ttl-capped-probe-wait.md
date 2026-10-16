# Card 9 — ttl-capped-probe-wait

**Edit:**
- `internal/creelengine#worker.run`
- `internal/creelengine_test#TestScheduler_Injects`
- `internal/creelengine/doc.go`
- `pattern/PATTERN-daemon-wakeups.md`
- `tools/sandbox/SANDBOX-CREEL-SUITE.md`

**Create:**
- `plan:internal/creelengine#Store.remainingTTL` -> `func (s *Store) remainingTTL(receiver string) (time.Duration, bool)`

**Uses:**
- `internal/creelengine#probeBackoffBase`
- `internal/creelengine#worker.probe`
- `internal/creelengine#Store.ageInjects`
- `internal/creelengine_test#env.probeWaits`
- `internal/creelengine_test#env.tick`
- `internal/creelengine_test#env.driveProbeWaits`
- `internal/creelengine_test#awaitOutcome`

**Intent:** Fix the expiry half of #686 (D8): an inject expires at its TTL, not at the next backed-off probe.

`Store.remainingTTL` returns the shortest TTL left, `TTL` minus `Consumed`, among the injects queued for the receiver, and false when none is queued; the probe that just ran charged every queued inject up to its time, and an inject posted since is charged from its posting.
In `worker.run`, after the backoff picks the next wait, a worker whose receiver was live and unpaused at its last probe caps that wait at `remainingTTL`, never below the one-second base, so the probe that ages the inject runs when its TTL runs out.
A dormant or paused receiver, whose TTL is frozen, and a receiver with no queued inject keep the plain backoff.
Bound: the cap only shortens a wait, only while an inject is queued and its TTL runs, and never below the one-second floor (`PATTERN-daemon-wakeups`).

Tests, rows of `TestScheduler_Injects` on the fake clock:
- "InjectExpiresWithinASecondOfItsTTLBetweenBackoffSteps": for TTLs of 10, 20 and 37 seconds, which fall between the probe times of the 1, 2, 4, 8, 8 backoff, an `InjectWait` into a never-idle alpha is driven by its recorded probe waits until it is answered, and it expires with the clock no more than one second past posting plus TTL; no recorded wait is under one second.
- The existing row "TTLIsFrozenWhilePausedAndWhileDormant" queues its inject with a 3-second TTL, short enough that a cap would show, and drives its waits probe by probe: under the pause they are 1, 2, 4, 8, 8 seconds, while dormant they stay at 8 seconds, and the inject is still queued in both and typed once the receiver is live and unpaused, so a frozen TTL keeps the plain backoff with no new row.
The existing row "InjectPostedMidWaitIsChargedFromItsPostingNotFromThePreviousProbe" keeps its assertions, since its 58-second tick still fires one pending probe and the capped one-second wait after it expires the inject at the next tick.

Docs in the same commit: the Delivery loop paragraph of `internal/creelengine/doc.go` and the creel worker line of `pattern/PATTERN-daemon-wakeups.md` say a live, unpaused receiver's wait is capped at its soonest queued inject's remaining TTL, never below one second; the C6 watch line of `tools/sandbox/SANDBOX-CREEL-SUITE.md` says a short `--ttl` into a never-idle pane expires within a second of its TTL, and into a bash strand names the stall `no-input-box`.

**ImpactSummary:** Only the delivery worker's wait between probes changes, and only downward while an inject's TTL runs; probing, aging and expiry logic and every signature are unchanged.
