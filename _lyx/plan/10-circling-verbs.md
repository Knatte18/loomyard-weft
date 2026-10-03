# Card 10 — circling-verbs

**Edit:**
- `internal/loomcli/review.go`
- `internal/loomcli/review_test.go`
- `internal/loomcli/cli.go`
- `docs/overview.md`

**Create:**
- `internal/loomrecipe/bouncerrows.go`
- `internal/loomrecipe/bouncerrows_test.go`
- `internal/loomcli/circling.go`
- `internal/loomcli/circling_test.go`

**Uses:**
- `internal/shedadapters/circling.go`
- `contracts/recipes/loom-recipe.yaml`
- `internal/loomengine/config.go`
- `internal/shedrun/paths.go`

**Intent:** Add `lyx loom circling accept [<slug>]` and `lyx loom circling continue [<slug>]`, the operator's two ways out of a circling halt (D7).

`loomrecipe/bouncerrows.go` adds `BouncerRunSubdir(row string) (string, bool, error)`: it parses the embedded loom recipe and returns the `run_subdir` of the named row when that row's engine is `Bouncer`, false for any other or unknown row, and a `loomrecipe:`-prefixed error on a parse failure.
Tests: each of the three loom Bouncer rows (named through the `loomshed` row constants) returns its run subdir, and a BurlerRound row, `PR-Gate` and an unknown name return false.

`review.go`: `resolveReviewTarget` takes the group name as a parameter, so its error text reads `loom: <group> <verb>` and its slug-form hint names the group, and both groups share it;
`review_test.go` passes `review`.
`cli.go`: the loom parent pre-run skips the circling group as it skips the review group, by generalizing `inReviewGroup` to cover both group names, since a slug must never be armed as a run-id;
the loom group's Long text and examples name the two verbs.

`loomcli/circling.go` builds the `circling` group with its own `PersistentPreRunE`, mirroring the review group: resolve the cwd, resolve the target worktree from the optional slug, and build a status reader over that worktree's self-run status file and lock.
The verb body, `circlingVerb(out, deps, decision)`, takes every side effect as an injected closure (read status, look up the Bouncer run subdir, record the decision) and refuses, each with a way-forward clause:

- no status file;
- a run that is not `awaiting`, or awaiting at a row `BouncerRunSubdir` does not report as a Bouncer;
- `shedadapters.ErrNotCircling`: the latest judged round's verdict is not `CIRCLING`;
- `shedadapters.ErrCirclingDecided`: a decision is already recorded for that round.

Otherwise it records the decision in `filepath.Join(loomengine.LoomReviewsDir(target), subdir)` through `shedadapters.RecordCirclingDecision` and returns an Ok envelope with the slug, the decision, the round and `"resume": "lyx loom start"`.
It never resumes the run itself, like `lyx loom approve`.
The verbs check no caller identity, like `lyx loom review` and `lyx loom approve`: any shell addressing the run can record a decision, an agent's tmux session in the task worktree included.
That is bounded by the refusal precondition alone — one decision per circling round, only while the run awaits at that Bouncer row — and by no agent stencil naming the verbs;
the decision file in the committed run directory records the decision for that round.
State this bound in the file header.

Tests in `circling_test.go`, over fake deps: accept and continue each record and return the round and resume hint;
a missing status file, a running run, a run awaiting at `PR-Gate`, a not-`CIRCLING` latest verdict and a second decision are each refused with exit 1 and record nothing new.
One test drives the cobra tree with a temp worktree to check the group's pre-run resolves a task worktree without a slug and that the parent pre-run never arms it.

`docs/overview.md`'s loom module line lists `circling accept|continue`, and its PR-Gate awaiting paragraph gains one sentence that a review segment's circling halt is also `awaiting`, resolved with these verbs.

**ImpactSummary:** Adds a new verb group under `lyx loom` and parametrizes the review group's target resolver, which only the review verbs and their tests call.

**Commit:** 10: loom: circling accept and continue resolve a circling halt
