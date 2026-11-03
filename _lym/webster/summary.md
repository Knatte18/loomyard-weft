# batten-remote-order: all five batches done

Every batch recorded `status: done` on the first `record-batch` call. No recovery, re-fork or plan fix ran. No digest carried `departures`, `renames` or `warnings`.

## Batch 01 — gitrepo-lease-and-retry (cards 1–4)

Head `05c069ac`.
gitrepo retries caller-named refusal reasons and returns `ErrLeaseRejected` from a leased delete.
It pushes a named branch and drops its tracking ref under the remote-refs lock.
`Add` drops a former pair's stale tracking ref and upstream, and re-adopts a live pair under an adopt-only option with typed refusals.
Deviations: `cmd/lym/gitrepoboundary_test.go`, `internal/fabricengine/add_adoptonly_integration_test.go`, `internal/fabricengine/add_remoteleftover_integration_test.go`, `internal/fabricengine/addadoptonly.go`, `internal/fabricengine/remoteleftover.go`, `internal/gitrepo/push.go`, `internal/gitrepo/pushbranch_integration_test.go`.

## Batch 05 — teardown-lock-add-discard-push (cards 5–6)

Head `4a394849`.
`Add`, the task-branch discard and synchronous pair pushes take the slug's teardown lock.
A teardown pushes the weft tip before its archive tag and archives a diverged origin tip.
Deviations: `internal/fabricengine/archive.go`, `internal/fabricengine/remove_archive_integration_test.go`, `internal/fabricengine/teardownlock_integration_test.go`.

## Batch 07 — leased-weft-delete (cards 7–9)

Head `e9238fdd`.
Every weft delete on origin is leased on the archived tip, and a moved branch is kept.
The cleanup sweeps try each slug's teardown lock and skip a held slug.
A run-scope reconcile may judge a run whose pair is absent.
A follow-up commit keeps the sweeps' lock file out of the mutation record, after the batch gate failed one cleanup test.
Deviations: `internal/fabricengine/cleanup.go`, `internal/fabricengine/cleanupremotewarp.go`, `internal/fabricengine/remove.go`.

## Batch 10 — batten-resume-readopt (cards 10–12)

Head `f930e4a7`.
A batten resume re-adopts a live pair whose task worktree is gone.
Fabric reads a slug's records tags and a records branch's file.
A pair that cannot be re-adopted fails the run once with a way forward by row.
Deviations: `internal/battencli/owner_readopt_test.go`, `internal/battencli/readopt.go`, `internal/battencli/readopt_integration_test.go`, `internal/battencli/readoptstop.go`, `internal/fabricengine/recordsreads.go`, `internal/fabricengine/recordsreads_integration_test.go`.

## Batch 13 — remove-follow-up (cards 13–14)

Head `c0417817`.
A removal of a batten run's pair names the step that settles its record.
The tackler reads the remote tip before judging a commit absent; its `git ls-remote` rule is static prose with no test, per `d5-no-prose-test`.
Deviations: `internal/battencli/cancel.go`, `internal/battencli/cancel_test.go`, `internal/battencli/resumereconcile.go`, `internal/landingshed/stuck.go`, `internal/pairteardown/followup.go`, `internal/pairteardown/followup_test.go`, `internal/pairteardown/teardown_test.go`, `internal/tendercli/deps.go`.

## Scope guard

Each digest listed deviations outside the batch's own targets, informational only; none made a batch stuck.
