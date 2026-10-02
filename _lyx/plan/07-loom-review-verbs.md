# Card 7 — `lyx loom review` verbs

**Create:**
- `internal/loomcli/review.go`
- `internal/loomcli/review_test.go`

**Edit:**
- `internal/loomcli#Command`
- `internal/loomcli/cli_test.go`

**Prosa:**
- `contracts/specs/refusal-spec.md`
- `docs/overview.md`

**Uses:**
- `plan:internal/parentreview#Store`
- `plan:internal/parentreview#Store.RecordDelivered`
- `plan:internal/parentreview#Store.AddNotify`
- `plan:internal/parentreview#Store.RecordVerdict`
- `internal/loomengine#LoomParentReviewDir`
- `internal/loomengine#LoomParentReviewLockDir`
- `internal/fabricengine#PrimeName`
- `internal/fabricengine#WorktreePath`
- `internal/loomcli#loomCLI.approveCmd`

**Intent:**
Add decision D6's `review` subtree under `lyx loom`: `notify [<slug>]`, `delivered [<slug>] [--failed <reason>]`, `approve [<slug>] [--review <file>]` and `reject [<slug>] <review-file>`.
They sit under `review` so they never collide with PR-Gate's `lyx loom approve`/`reject`.

The `review` group carries its own `PersistentPreRunE`, so the loom parent's `arm` never runs for it and never reads a slug as a run-id.
It resolves the cwd through `lyxcwd`, then the target worktree: a given slug resolves through `fabricengine.WorktreePath` and `lyxcwd.ResolveWorktree` from any worktree of the hub, and a missing worktree is the "unknown slug" refusal; with no slug, the prime (`location.WorktreeName == fabricengine.PrimeName(location)`) refuses "slug required from the prime", and a task worktree addresses itself.
It builds a `parentreview.Store` from the target's `LoomParentReviewDir`/`LoomParentReviewLockDir`.
`reject`'s review-file argument and `approve`'s `--review` file both resolve against the invoking cwd.

The verb bodies take their side effects as injected deps, following `approveVerb`, so every refusal is reachable from an untagged test.
Each maps the store's sentinels onto messages ending in a `way forward:` clause:
no open request names that only Discussion-Write's gate opens one and points at `lyx loom status <run>`;
verdict already recorded names that the round is settled and the run proceeds on its own;
expired names that the run passed on its wait bound and Discussion-Review still reviews it;
a missing or empty review file names writing the review and re-running `reject`;
slug required and unknown slug name the slug form and `lyx board list`.
Every command has a non-empty `Short`, JSON errors through `internal/output`, and a `clihelp.ShouldAbort` check first.
The verbs check no caller identity.

Bound: an unchecked verb reaches no further than this one advisory review — an `approve` skips at most the pass-on-cap parent review, a `reject` spends at most its single re-prompt, a `delivered` stamped without a send stops re-sends but leaves the wait visible in status until the bound, and a `notify` sends one prompt and changes no gate count; Discussion-Review runs after every outcome.

`review_test.go` covers each verb's success write and one test per refusal: no open request, verdict already recorded (`notify`, `approve`, `reject`), expired (all four), missing or empty review file, slug required from the prime, unknown slug.
`cli_test.go`'s exact-set guard gains `review`, and the help-tree check covers its four children.
`refusal-spec.md`'s loom section gains one row per refusal, quoting each way-forward clause.
`docs/overview.md`'s loom row lists `review` and names `internal/parentreview`, its `loom.yaml` key list gains `parent_review_wait_min`, and its `status` paragraph gains one sentence on the waiting-on-reviewer note; the package tree gains an `internal/parentreview/` line.
`Command`'s `Long` text and examples describe the subtree.

**ImpactSummary:** One new cobra subtree registered on the loom command; existing verbs, their arming and the PR-Gate `approve`/`reject` are untouched.
