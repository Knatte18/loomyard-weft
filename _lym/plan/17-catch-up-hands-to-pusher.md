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

**Intent:** D8: the dobby's catch-up hands an owed push to the pusher and starts the run.

`pushOwedForCatchUp` takes its reads, its push and its pusher start as the `catchUpPushSeams` fields `Pair` (the run's pair and whether it stands here), `Held` (whether a landing holds the run's records), `Push` and `Start`;
`HubCatchUp` fills them with `catchUpPair`, `pushledger.HeldRecordsCommit`, `pairteardown.PushRun` with today's `RunPush`, and `runpusher.Start`.
When the prime's records push fails as owed, by `fabricengine.IsPushOwed`, a held push lock included, it starts the prime's pusher and goes on to the pair;
when the pair's push fails as owed it starts the pair's pusher; either returns nil, so the catch-up starts the run.
A records commit a landing entry holds is not pushed (card 9): when `Held` reads the run held, the commits held back are owed to the pair's pusher, which it starts.
A refusal still fails the catch-up with its way forward, and a push that hands the run over still reads as done.
The catch-up's fetch stays the owner verdict PATTERN-owner-machine requires, so its failure still stops the run's start (Scope, Out).
That a nil `PushOwed` starts the run is the engine's, pinned by `TestCatchUp` in `internal/dobbyengine`.

`TestPushOwedForCatchUp`, tier0, red-first, adding under a second to one tier0 run, over fake seams that spawn nothing:
a pair push failing with `fabricengine.ErrPushLockBusy` starts the pair's pusher and returns nil, where the merge base fails the catch-up;
a prime push failing owed starts the prime's pusher and still pushes the pair;
a refused push returns its error and starts nothing;
a held run starts the pair's pusher after its push;
and a run with no pair here pushes the prime's records alone.

**ImpactSummary:** Re-signs one unexported catch-up step, whose one caller is `HubCatchUp`; the catch-up's other steps are unchanged.
