# Card 7 — circling-decision-record

**Create:**
- `internal/shedadapters/circling.go`
- `internal/shedadapters/circling_test.go`

**Edit:**
- `internal/shedadapters/round.go`

**Uses:**
- `internal/shedadapters/bouncerfiles.go`
- `internal/shedadapters/burler.go`

**Intent:** Define the operator's recorded decision on a circling round (D7) as a file contract the CLI writes and the Bouncer reads, with the refusal precondition inside the writer.

`round.go` gains `circlingDecisionPath(runDir, round)`, naming `round-<N>-circling-decision.md` in the run directory, so the decision is committed with the run like the ledger.

`circling.go`:

- An exported `CirclingDecision` string type with `CirclingAccept` (`accept`) and `CirclingContinue` (`continue`).
- The file is `---`-delimited YAML frontmatter carrying `round`, `decision` and `settled` (bool), parsed fail-loud through `splitFrontmatter` like the other Bouncer files: the round must match the filename, the decision must be one of the two values, and `settled: true` is legal only on an accept.
- Exported sentinels `ErrNotCircling` and `ErrCirclingDecided`.
- `RecordCirclingDecision(runDir string, d CirclingDecision) (int, error)` resolves the latest round through `ResolveRound` with the pinned `round-<N>-review.md` name (`roundReviewPath`'s convention), requires that round's `recordedVerdict` to be `verdictCircling` (else `ErrNotCircling`), and creates the decision file with `os.O_EXCL` as pending (`settled: false`), returning `ErrCirclingDecided` when a file for that round already exists.
  It returns the round it recorded.
- `readCirclingDecision(runDir, round)` returns the decision, whether it is settled, and whether a file exists; a present but malformed file is an error.
- `settleCirclingAccept(runDir, round)` rewrites a pending accept as settled, so the Bouncer can record the settle before it runs the Approve seam.

The writer's checks are the whole mechanical bound on the verbs that card 10 adds: a decision exists only for a round whose judged verdict is `CIRCLING`, and only once per round.
The run-state half of the precondition (awaiting at a Bouncer row) belongs to the verbs, which read the status file.

Tests in `circling_test.go`, over a temp run directory:
recording accept and continue on a `CIRCLING` round writes a pending file that `readCirclingDecision` reads back with the right round;
a second record for the same round returns `ErrCirclingDecided` and leaves the first file byte-identical;
a latest verdict of `CONVERGED` or `CONTINUE`, an unjudged latest round, and an empty run directory each return `ErrNotCircling` and write nothing;
`settleCirclingAccept` turns a pending accept into a settled one;
a decision file whose round disagrees with its filename, an unknown decision, and a settled continue are each read as errors.

**ImpactSummary:** Adds one path helper in `round.go`; the Bouncer reads the record from card 8 and the CLI writes it from card 10.

**Commit:** 7: bouncer: record an operator decision on a circling round
