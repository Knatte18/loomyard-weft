# Card 4 — pairteardown-prime-handover

**Edit:**
- `internal/pairteardown#RunPush`
- `internal/pairteardown#PushRun`
- `internal/pairteardown#HandOver`
- `internal/pairteardown/doc.go`
- `internal/pairteardown_test#TestPushRun_HandsOverOnATakeover`

**Uses:**
- `internal/fabricengine#HandOverTips`
- `plan:internal/fabricengine#TakeoversError`
- `plan:internal/fabricengine#ErrTakeoverPassOwed`

**Intent:**
Hand over each batten run a prime push meets taken over, and then push the rest (D6).

- `RunPush` gains `PrimeRecords bool`.
  `HandOver` sets `HandOverRequest.PrimeRecords` to the prime when it is set,
  so the run's unpushed prime commits go under an owner archive tag before its hand-over mark (card 3).
- `PushRun`, on a push that returns a `*fabricengine.TakeoversError`, hands each pending run over through `HandOver`
  with `InsidePair` false and `PrimeRecords` true, which removes that run's task pair as batten's hand-over does today,
  and then pushes again; each later push meets the written marks and leaves those runs' commits out.
  A failed hand-over returns an error wrapping `fabricengine.ErrTakeoverPassOwed` naming the run and the step,
  with nothing pushed, so the pusher retries the whole pass in its backoff and the retried pass hands the run over again.
  It stops pushing again once a push returns anything but a `*TakeoversError`, and returns that push's result.
  A hand-over's own `ErrHandedOver` return is the expected end of each hand-over and is not a failure.
- The pair path keeps today's single takeover and its hand-over.

`pairteardown/doc.go`'s `PushRun` paragraph says it is the push the pusher, the teardown and the catch-up run,
and names the prime's many-run hand-over.

`TestPushRun_HandsOverOnATakeover` is tier1 and gains prime rows,
each with two batten runs' unpushed prime commits and one run taken over on origin:
- `PushRun(prime, prime, "", ...)` returns nil, the taken-over run's hand-over mark stands, its owner archive tag holds the prime's tip,
  its pair is removed, and origin's prime records hold the other run's commits in order and none of the taken-over run's;
- with a pre-receive hook on origin refusing tags, the hand-over's archive fails:
  an error wrapping `ErrTakeoverPassOwed` that `IsPushOwed` reads owed, no hand-over mark, and origin's prime records unchanged;
  once the hook is removed, a second `PushRun` hands the run over and lands the other run's commits.

**ImpactSummary:** PushRun gains one branch for the prime's many-run takeover; every caller that passes a run slug keeps its path.
