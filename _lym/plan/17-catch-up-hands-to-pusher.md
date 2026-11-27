# Card 17 — catch-up-hands-to-pusher

**Edit:**
- `internal/battencli#pushOwedForCatchUp` -> `func pushOwedForCatchUp(prime *lymcwd.Location, slug string, seams catchUpPushSeams) error`
- `internal/battencli#HubCatchUp`

**Create:**
- `plan:internal/battencli#catchUpPushSeams` -> `type catchUpPushSeams struct`
- `plan:internal/battencli#TestPushOwedForCatchUp` -> `func TestPushOwedForCatchUp(t *testing.T)`

**Uses:**
- `internal/battencli#catchUpPair`
- `internal/battencli#handedOverIsDone`
- `internal/battencli#TestHubCatchUp`
- `internal/dobbyengine#TestCatchUp`
- `plan:internal/pushledger#HeldRecordsCommit`
- `plan:internal/pushledger#Update`
- `plan:internal/fabricengine#HoldPairPushLock`

**Intent:** D8: the dobby's catch-up hands an owed push to the pusher and starts the run.

`pushOwedForCatchUp` takes its reads, its push and its pusher start as the `catchUpPushSeams` fields `Pair` (the run's pair and whether it stands here), `Held` (whether a landing holds the run's records), `Push`, `Start`, `LockWait`, the push-lock wait it puts on its `RunPush`, and `BackoffSleep`, the backoff sleep it puts on its `RunPush` (card 9);
`HubCatchUp` fills them with `catchUpPair`, `pushledger.HeldRecordsCommit`, `pairteardown.PushRun`, `runpusher.Start`, `fabriclayout.StatusPushLockWait` and a nil sleep, the production backoff.
When the prime's records push fails as owed, by `fabricengine.IsPushOwed`, a held push lock included, it starts the prime's pusher and goes on to the pair;
when the pair's push fails as owed it starts the pair's pusher; either returns nil, so the catch-up starts the run.
A records commit a landing entry holds is not pushed (card 9): when `Held` reads the run held, the commits held back are owed to the pair's pusher, which it starts.
A refusal still fails the catch-up with its way forward, and a push that hands the run over still reads as done.
The catch-up's fetch stays the owner verdict PATTERN-owner-machine requires, so its failure still stops the run's start (Scope, Out).
That a nil `PushOwed` starts the run is the engine's, pinned by `TestCatchUp` in `internal/dobbyengine`.

`TestPushOwedForCatchUp`, tier1, red-first, over a `fabrickit` hub fixture with a pair and a bare origin, adding about two seconds to one tier1 run.
Its seams are `HubCatchUp`'s real ones but three: `LockWait` is a few milliseconds, so a held push lock fails at once instead of after the production wait,
`BackoffSleep` returns at once, so the unreachable-origin and refusal rows run no production retry delay,
and `Start` is a wrapper that records each target and passes it to the real `runpusher.Start`, which spawns nothing because the test holds each target's pusher lock.
It pins:
the pair's weft-side push lock held by the test through `fabricengine.HoldPairPushLock` (card 12), so the pair push fails with `fabricengine.ErrPushLockBusy`: the pair's target is handed to `Start` and the call returns nil, where the merge base fails the catch-up;
an unreachable prime origin, its URL repointed by `git remote set-url` run through `gitkit.Git`: the prime's target is handed to `Start` and the pair is still pushed;
origin's records branch moved on by another clone, a refusal: the error returns and nothing is handed to `Start`;
a run held by a landing entry written through `pushledger.Update`: the pair's target is handed to `Start` after its push, and origin lacks the held commit;
and a run with no pair here: the prime's records alone are pushed.

**ImpactSummary:** Re-signs one unexported catch-up step, whose one caller is `HubCatchUp`; the catch-up's other steps are unchanged.
