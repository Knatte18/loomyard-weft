# Card 15 — batten-teardown-records-remedy

**Edit:**
- `internal/battencli#battenCLI.wire`
- `internal/battencli/wire_test.go`

**Prosa:**
- `docs/overview.md`

**Uses:**
- `plan:internal/fabricengine#ErrPairSiblingDirty`
- `internal/fabricengine#RemoveResult`
- `internal/fabricengine#PairBranchResult`
- `internal/battencli#createRefusal`

**Intent:**
When batten's teardown finds the task's run records uncommitted, its Stuck reason names batten's own recovery, never fabric's `--force`, which would discard exactly the records this task exists to keep.

Add `teardownRefusal(err error, slug, taskAnchor string) error` to `wire.go`, shaped like `createRefusal`: when `errors.Is(err, fabricengine.ErrPairSiblingDirty)` it returns a new error saying the task worktree's run records are uncommitted (most likely a driver ended before its end-of-session commit), and naming the recovery: run `lyx loom commit-records` in `taskAnchor`, then resume this run with `lyx batten run <slug>`; it names no `--force` and no fabric command.
Every other error passes through unchanged.
The `Teardown.Remove` closure resolves the task worktree's anchor before calling `Topology.Remove`, passes Remove's error through `teardownRefusal`, and adds `archive_tag` and `archive_skipped_reason` to its existing teardown log line; the already-gone arm logs `RemovePairBranch`'s archive fields the same way.
The wording stays inside battencli's vocabulary: "the task worktree" and "the pair", never either fabric side's name.

In `docs/overview.md`'s batten section, the teardown sentence that gives "fabric's own refusal as the `stuck_reason` — the operator cleans the child and re-steps" says that uncommitted run records in the task worktree instead name batten's own recovery (`lyx loom commit-records` there, then resume the batten run), other uncommitted content keeps fabric's refusal, and neither is ever `--force`.
The paragraph on teardown's branch deletion adds, in that paragraph's own "other-side branch" wording, that the removal first pushes an `archive/<slug>/<tip>` tag of that branch's tip to its remote, so the run's committed records stay reachable after the branch is gone.

Tests in `wire_test.go`: `teardownRefusal` over an error wrapping `fabricengine.ErrPairSiblingDirty` names `lyx loom commit-records`, the anchor path and `lyx batten run <slug>`, and does not contain `--force`; an unrelated error comes back unchanged.

**ImpactSummary:** Rewords one teardown refusal and extends two log lines in battencli's Remove closure; the teardown sequence is unchanged.
