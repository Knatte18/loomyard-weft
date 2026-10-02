# Card 10 — review-approve-supersede

**Edit:**
- `internal/loomcli/review.go`
- `internal/loomcli/review_test.go`

**Prosa:**
- `contracts/specs/refusal-spec.md`

**Uses:**
- `plan:internal/parentreview#Store.SupersedeCapReject`
- `plan:internal/parentreview#ErrNotAtCap`
- `internal/shedrun#StatusFile`
- `internal/shedrun#StatusLock`
- `internal/state#ReadJSONStrict`

**Intent:**
Make `lyx loom review approve` the way out of a run halted at the reject cap (Decision 5), and state the new bounds of `approve` and `reject` in their help text.

`reviewApproveVerb` gains a run-status reader, a `func() (shedengine.Status, bool, error)` the pre-run builds over the target worktree's default run (`state.ReadJSONStrict[shedengine.Status]` at `shedrun.StatusFile`/`shedrun.StatusLock` for `shedrun.SelfRunID`), injected so tests pass a fake.
When the latest round carries a reject verdict, approve takes the supersede path:

- it refuses unless the run's status is `blocked`, with a message saying approve supersedes a reject only on a run halted at the reject cap, and naming the way forward (`lyx loom status <slug>`; below the cap the gate re-prompts the writer itself, and the next round's request is approved normally);
- it refuses a given `--review` on that path, with the way forward of re-running without `--review`, since the cap's review file is kept;
- it calls `Store.SupersedeCapReject` and maps `ErrNotAtCap` to a refusal saying the latest round's reject is not the cap's, with the same way forward;
- on success the envelope carries `"action": "approve"` and `"superseded": true`, and the message's way forward is `lyx loom start` in the task worktree.

Every other approve keeps today's `RecordVerdict` path and refusals.

Bound: approve, like reject, checks no caller identity, so the halted state (status `blocked`, the store at the cap, the latest round the cap's reject) is the only gate on the supersede path, and it never rewrites an earlier round or an open round's pending state.
`reject` can now halt a run, but only on the cap's rejected round and only into `blocked`, which one approve lifts.

Rewrite `review.go`'s file comment paragraph on unchecked verbs, which says an approve skips at most a pass-on-cap review and a reject spends one re-prompt, and the `review` group's, `approve`'s and `reject`'s `Short`/`Long` help so they state those bounds: the parent reviews every rewrite after a reject, the cap's reject halts the run `blocked`, and approve on that halted run supersedes the reject.
Add rows to `contracts/specs/refusal-spec.md`'s loom review section for the three new refusals (run not halted at the cap, latest round not the cap's reject, `--review` on the supersede path), per the Refusal Way-Forward Invariant.

Tests in `review_test.go`, each refusal reached through the verb with a temp store and a fake status reader: approve at the cap on a `blocked` run supersedes, keeps `review.md` and reports `superseded`; on a `running` run it refuses naming the way forward; with the latest reject below the cap it refuses with `ErrNotAtCap`'s message; with `--review` on that path it refuses; an ordinary approve on an open round still records a plain approve.

**ImpactSummary:** Adds a supersede path to `lyx loom review approve` and rewrites the review verbs' help; notify, delivered and reject verb bodies are unchanged.
