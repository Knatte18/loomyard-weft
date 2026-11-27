# Card 13 — finalize-pr-owed-check

**Edit:**
- `internal/landingshed#Finalize.landUnderLock`
- `internal/landingshed#Finalize.pullRequestLanding`
- `internal/landingshed#TestFinalize_PullRequest`

**Create:**
- `plan:internal/landingshed#Finalize.parentPushBlocks` -> `func (fz *Finalize) parentPushBlocks(ctx context.Context) *ending`
- `plan:internal/loomcli#TestLandingDeps_ParentPushState` -> `func TestLandingDeps_ParentPushState(t *testing.T)`

**Uses:**
- `internal/landingshed/doc.go`
- `plan:internal/landingshed#ParentPush`
- `internal/runpusher#RecordLanding`
- `internal/fabricengine#RecordPushFailure`
- `internal/fabriclayout#PushFailureHeldBack`

**Intent:** D5: a landing that needs GitHub never merges onto a parent whose landing push origin still lacks, so GitHub never merges into a stale base.

`parentPushBlocks` reads `Deps.ParentPushState`: an owed parent push returns a `git-transport` transient error, which the Go driver waits out (D6);
a refused one is a blocked halt naming the refusal's way forward and ending with `lym loom resume` in the task worktree, which lands once the way forward has cleared the record;
neither returns nil, as does a nil seam.
`landUnderLock` runs it before it takes the parent lock when the landing needs GitHub as the disk reads it lock-free:
the run's pull-request record or the beam's landing record names a pull request, or `PullRequest.Mode` reads `trail` or `review`.
A landing whose GitHub need only the parent's rules read reveals is checked under the lock alone.
Under the lock, once `pullRequestLanding` has decided the landing merges on GitHub (`landing.merge`), it runs it again;
finding the push owed it releases the lock and returns the transient error, so a local landing recorded while it waited for the lock never leaves GitHub merging onto a stale base.
A local landing never runs it: it lands on an owed parent and adds its own entry (card 12).

Tests.
The rows that need a filled `ParentPushState` drive it through the real `landingDeps` wiring, so no row stands in for `runpusher` or the ledger.
`TestLandingDeps_ParentPushState`, tier1, in `internal/loomcli`, over a `fabrickit` hub fixture with a run at Finalize, adding about eight seconds to one tier1 run, runs Finalize through `landingDeps`:
an open recorded pull request, answered through `prkit`, with a landing on the parent target written through `runpusher.RecordLanding`, returns a `git-transport` error before the lock is taken and merges nothing;
the same with the prime code branch's `held_back` record written through `fabricengine.RecordPushFailure` halts naming the refusal's way forward and `lym loom resume`;
a trail landing with no record and the push owed returns the error before the lock;
a trail landing whose landing entry the test records while it holds the parent lock returns the error with the lock released and no GitHub merge;
and a local landing with the push owed lands.
`TestFinalize_PullRequest` gains one tier0 row, adding under a second to one tier0 run: a nil `ParentPushState` lands as today.

**ImpactSummary:** Adds one check to the pull-request path of Finalize; a local landing is unchanged.
