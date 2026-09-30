# Card 40 — loom-approve-way-forward

**Edit:**
- `internal/loomcli/approve.go`
- `internal/loomcli/approve_test.go`
- `internal/loomcli/gotoentry_test.go`

**Uses:**
- `internal/landingshed/prgate.go`
- `internal/loomshed/loomshed.go`

**Intent:**
Fix PR #321 findings 4a and 4b: three `lyx loom approve` refusals name a way forward that does not work where the refusal is reached.

- 4a, HEAD differs: `push or sync the task branch (lyx fabric push)` is wrong, because `lyx fabric push` pushes only the `_lyx` side while the pull request's head is the code task branch.
  The way forward becomes `way forward: push the task branch with "git push" when the local HEAD is ahead, or pull it with "git pull" when the pull request's head is ahead, then re-run lyx loom approve`.
- 4b, no pull request and pull request not open: `lyx loom step opens it at Publish` loops, since the run sits at PR-Gate, which returns awaiting when there is no pull request or it is closed, so a step never reaches Publish.
  Both way forwards become `way forward: "lyx loom goto --to Publish" moves the run back to Publish, then "lyx loom step" opens a new pull request`, a backward goto card 33 admits from a run awaiting at PR-Gate.

The row name comes from `loomshed.NamePublish`, as the not-at-PR-Gate text already takes `loomshed.NamePRGate`.

Tests:

- `approve_test.go`: the HEAD-differs refusal names `git push` and `git pull` and no longer names `lyx fabric push`; the no-pull-request and not-open refusals name `lyx loom goto --to Publish` and no longer name `opens it at Publish`.
- `gotoentry_test.go`: a loom status file awaiting at PR-Gate, written as the verbs write it, takes `goto --to Publish` and is left paused at Publish, with the goto history entry; `goto --to Finalize` on the same file is refused.

**ImpactSummary:** Three approve refusal texts change; approve's checks, order and envelope keys are unchanged.
