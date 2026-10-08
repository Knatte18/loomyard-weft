# Driver repair 1: Publish merge-in

Failure: `landingshed: Publish: merge-in against parent branch "main": mergeresolve: list untracked files (attempt 1): fabricengine: list untracked files: gitrepo: worktree status: object not found` (kind `producer`, transient empty).

Trace lines acted on (trace 82c70335c18c4f83):
- `fabric: mutation kind=merge_staged target=binary-change-and-rules detail=7b3d140c...`
- conflict agent shuttle run finished, outcome=done
- `shed: step refused kind=producer`

Diagnosis: a merge is staged and `internal/battencli/wire.go` is unmerged with no conflict markers left; `git status` and `git cat-file` on all three stage blobs succeed now, so the `object not found` was not a persistent corruption.
No `lyx` repair verb covers a staged merge-in.

Action: re-step once so the resolver re-reads the staged merge.
Outcome: see the stop report.
