# Card 9 — inject-ttl-from-posting

**Edit:**
- `internal/creelengine#Store.ageInjects` -> `func (s *Store) ageInjects(receiver string, since, now time.Time) error`
- `internal/creelengine#worker.probe`
- `internal/creelengine#worker.finish`
- `internal/creelengine_test#TestScheduler_Injects`

**Uses:**
- `internal/creelengine_test#env`
- `internal/creelengine_test#newEnv`
- `internal/creelengine_test#env.start`
- `internal/creelengine_test#env.tick`
- `internal/creelengine_test#env.probeWaits`
- `internal/creelengine_test#awaitOutcome`
- `internal/creelengine_test#journalText`

**Intent:** Decision D6: each queued inject is charged TTL only from its own posting.

`Store.ageInjects(receiver, since, now)` charges every inject queued for the receiver `now` minus the later of `since` and the inject's posting time, clamped at zero, and expires those that used up their TTL as today.
The posting time is the entry's `Posted`, the daemon Clock's time at the append of its `posted` record, which `pool.applyPosted` already keeps; a carried inject keeps its journaled consumed TTL and its original posting time, so it is charged as before.
`worker.probe` passes its previous probe's time and now, and `worker.finish` its previous probe's time and the clock's now, so the TTL counted at the finish follows the same rule.
The first probe after a start still charges nothing, since `known` is false then, so a restarted daemon charges no inject for the time it was down; `JournalOfAKilledDaemonRebuildsTheQueuedInjectWithItsConsumedTTL` in `TestDaemon_Lifecycle` already pins that and stays green.

`TestScheduler_Injects` gains a row where a busy receiver's probes back off to the 8 s ceiling and a 60 s inject is posted 7 s into one wait: the inject is still queued 59 s after its posting and expires once 60 s have passed since it, never 7 s early as the old charge from the previous probe made it.
A second row pins the finish: a send blocked with `SetSendFunc` holds the worker inside a probe of a first inject, the clock moves 30 s, a second inject is posted, the clock moves 5 s, and the scheduler is stopped while the send is released, so the worker's finish ages the second inject; the journal then holds its `aged` record with `consumed_s` 5, the time since its posting, never 35.

**ImpactSummary:** One re-signed unexported Store method with its two callers in deliver.go; aging is unchanged for injects queued before the previous probe.
