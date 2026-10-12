# Merge-review perch and a parent lock that survives sleep and crashes

## What changes and why

When a run merges its parent branch in, no review has seen how the task's code interacts with what the parent gained meanwhile; verify catches only what `go test` and PATTERN's scans enforce.
A cold-start agent now reviews that interaction, and a per-parent lock keeps any other run from moving the parent until the run lands or stops making progress.
Everything works for any parent branch, never a fixed one.

- **`Merge-In` row.**
  The parent merge-in, its conflict session and its post-merge plan verify move out of Publish into a new `landingshed` producer that runs for every parent, including one outside `require_pr_to_base`.
  Publish keeps `publish_verify`, the push and the pull request.
  Loom's order is now `Webster-Bouncer → Describe → Merge-In → Merge-Bouncer → Publish → PR-Gate → Finalize`; darn's recipe gains `Merge-In` without the merge perch.
  Under the parent lock, `Merge-In` fetches the parent's upstream and fast-forwards the local parent pair before merging, and writes a merge record (merged parent tip, tip the last finished review reached, task head, overlap).
  A local parent ahead of or diverged from its upstream is Stuck with a way forward; `Merge-In` never pushes or rewrites the parent.
- **`Merge-Review` perch.**
  A `Merge-Bouncer` / `Merge-Burler-Review` / `Merge-Burler-Fix` segment in loom's recipe, with the new `loom-rubric-merge-review` stencil judging only the interaction between the parent's new commits and the task's code.
  A skip seam runs the round only when the parent commits since the last finished review overlap the task's change by file and Go package, with no agent spawned otherwise.
  `loom.yaml` gains `merge_max_bounces: 1`, `merge_auto_accept: true` and `skip_merge_review: false`.
  Auto-accept on a spent budget escalates instead when an open BLOCKING finding remains.
- **Parent lock (`internal/parentlock`).**
  One record file per parent branch at hub level, owned by the run's detached shed loop process and valid while that process lives, never by wall-clock time.
  A host that sleeps keeps the lock and a crashed owner frees it at once.
  Waiting runs queue by ticket and show as waiting in `lyx loom status`.
  The lock is released on any stop, and the owning loop also releases it when its agent sits in a waiting state past `parent_lock_idle_release_min` (landing.yaml), measured on the monotonic clock, which excludes suspend on Linux.
- **Moved-parent reroute.**
  `shedengine` gains an `on_reroute` recipe edge, capped, with a halting Stuck.
  Publish, PR-Gate and Finalize reroute to `Merge-In` when the parent moved outside the run's merge record.
  Batten resumes a child awaiting at PR-Gate whose parent moved, and holds a PR-Gate lock as a backstop for a release its owner missed.
- **Verbs.**
  `lyx batten lock list` and `lyx batten lock release <parent-branch>` / `--all`; the release-all function is what `lyx pause` (note `lyx-halt`) will call.
- **Orch stencil.**
  The PR-Gate fork now judges the run against its board goal and the operator's decisions, leaving code review to the Webster and merge reviews.
- The Publish verify failure record and batten's Publish retry name the row that ran the failing verify (`Merge-In` or `Publish`).
- Docs: `doc.go` for landingshed, battenshed, shedengine, shedverbs and `parentlock`, `docs/overview.md`, `PATTERN.md` with its touched `pattern/` files, and the loom status and refusal specs.

Closes: none; the board entry links no issue.

## Design decisions

- The owner is the run's detached `--until-stop` loop, read from its pid record; a step run outside a loop names its own process, so the lock lapses when that step exits.
- No lock verb exists for a hand landing outside a run: a hand landing goes through `lyx loom goto --to Finalize`, and a raw push to the parent is caught as a moved parent.
- Batten's Land row takes no parent lock; Finalize has already landed and released it.
- A run parked on a row whose name or edges changed is restarted, not migrated, as with earlier recipe changes.
- A pair whose `loom.yaml` lacks the new keys reads zero values until `lyx config reconcile`: the review runs with `review_max_bounces` as its budget and no auto-accept.
- `LoadConfig` fills absent merge keys from the template, so the implementation keeps that behaviour where a card described zero values.

## To check by hand

- The pair fast-forward in `Merge-In` pulls the parent pair's worktree; check that the weft step in that fast-forward is handled as intended (`internal/fabricengine`).
- The merged parent tip is recorded from the upstream; check it is read after the fetch, not before (`internal/landingshed`).
- Native Windows behaviour of the monotonic-clock idle bound is unverified; WSL runs the Linux kernel.
- A live but hung owner keeps the lock until the idle bound, batten's backstop or `lyx batten lock release`; detecting the hang is task `driver-arbiter`'s.
- `merge_max_bounces`, `merge_auto_accept`, `skip_merge_review` and `parent_lock_idle_release_min` need classification in `config-layers-run-settings` whichever task lands second; `driver-arbiter` must likewise adopt `Merge-In` as the row that owns the plan-verify halt.
- The integration tier (`lyx gate test --tags integration ./...`) was not run for this description; run it before pushing.
