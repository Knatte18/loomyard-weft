# Relevant files

- `internal/battencli/commitstatus.go` `newCommitStatusSeam`, `commitStatusMarker` — batten's transition seam with the start-row exception and `push_pending` marker
- `internal/loomcli/wiring.go` `newCommitStatusSeam` — loom's transition seam, with the run-scope origin reconcile before commit and push
- `internal/statuscommit/statuscommit.go` `New`, `retryOwedPush` — the shared commit-then-push core and its owed backoff
- `internal/pairteardown/handover.go` `PushRun`, `HandOver` — the transition push and the hand-over on another machine's takeover
- `internal/fabricengine/pushanchored.go` `PushPairAnchored` — records-then-code pair push under the push lock
- `internal/fabricengine/pushclassified.go` — rejected records push: fetch, takeover check and re-push
- `internal/fabricengine/spawn.go` `SpawnStatusPush` — existing detached push of the status records
- `internal/fabricengine/coalesce.go` — order of the detached pushes
- `internal/shedengine/run.go` — every status write calls the `CommitStatus` hook, skipped rows included
- `internal/adoptcli/foreground.go` — adopt's takeover over origin's run status
- `internal/fabricengine/takeoverwrite.go` — writes the takeover entry and moves local branches onto origin's tips
- `internal/landingshed/prreview.go` — PR-Review's find-or-open of the pull request
- `internal/landingshed/finalize.go` — Finalize's guards against a repeated landing
- `contracts/recipes/batten-recipe.yaml` — batten's rows after Run-Shed: Land and Worktree-Teardown
- `contracts/recipes/tapestry-recipe.yaml` — loom rows a run walks, skipped ones included
