# Review parent merges and lock the parent from merge to landing

When a run merges its parent branch in, a cold-start agent now reviews how the task's code interacts with what the parent gained meanwhile.
No other run can merge into or land on that parent until the run lands or stops making progress.
Until now the PR-Gate fork reviewed the whole diff, Webster had reviewed only the task's own code, and nothing reviewed the interaction between the two; verify catches only what `go test` and PATTERN's scans enforce.
Everything works for any parent branch, never a fixed one.

## What changes

- **`Merge-In` row.**
  Publish no longer merges the parent in; the new `Merge-In` row does, in loom and darn.
  It takes the parent lock, fast-forwards the parent pair when it is behind its upstream (Stuck when that pair is dirty, mid-merge or missing), merges the parent's tip and writes a merge record.
  Under the told push skip it merges the local tip alone.
- **`Merge-Review` perch (loom only).**
  A `Merge-Bouncer` skip seam runs the round only when the parent commits since the last finished merge review share a file or a Go package with the task's own change, or change `PATTERN.md` or `pattern/`.
  An up-to-date merge-in skips.
  A missing or unreadable merge record falls back to a real review of the whole merge-in diff.
  The default is one round, with auto-accept on the cap; a BLOCKING finding the fix cannot settle escalates instead of auto-accepting.
  `skip_merge_review` turns the perch off; new loom.yaml keys carry the round budget and the auto-accept setting.
  The reviewer is cold and reads only the overlap, against `PATTERN.md` and the task's code (`loom-rubric-merge-review` stencil).
- **Parent lock (`internal/parentlock`).**
  One record per parent branch lives under the board dir's `.lyx` tree, so every worktree on the hub sees it.
  It names the owner run and the owner process by pid and start time, so validity follows the process, never wall-clock time: a host that sleeps keeps the lock, and a crashed owner frees it at once.
  Waiters queue by ticket, oldest live ticket first, polling from one second backing off to thirty.
  An unreadable record is Stuck with a way forward naming `lyx batten lock release <branch>`.
- **Release paths.**
  The owner is the run's detached `--until-stop` loop; its exit hook releases the lock at every stop kind, by compare-and-release on its own pid and start time.
  An awaiting stop at PR-Gate under a live batten hands the record to batten instead, so the lock holds while the orch reviews.
  Finalize releases on Done, before Friction-Reflect, and a rejected PR-Gate releases before PR-Rework (darn: Darn-Write).
  The loop also releases when its run's agents have all been waiting longer than `parent_lock_idle_release_min` (hub-wide `landing.yaml`, default 10, measured on the process's monotonic clock so suspend never counts).
  Batten backstops a handed-over or stalled-owner record at its notice-probe cadence and logs one Warn per release.
- **Moved-parent reroute.**
  `shedengine` gains a recipe-declared `on_reroute` edge and a history-only `reroute` outcome.
  `Publish`, `PR-Gate` and `Finalize` in loom and darn declare `on_reroute: Merge-In`.
  Each first ensures the lock, then compares the local parent tip and its upstream tip (after a fetch) with the tip the merge record says was merged; if either differs and the parent does not already hold the task's work, the run goes back to `Merge-In`.
  A row reroutes at most three times before a blocked halt for a human; `lyx loom resume` grants three more.
  Batten resumes a child awaiting at PR-Gate whose parent moved, with no notice, so the orch never reviews a pull request whose base moved.
- **Verbs and visibility.**
  `lyx batten lock list`, `lyx batten lock release <parent-branch>` and `lyx batten lock release --all`; the engine exposes the release-all function `lyx pause` will call.
  A run waiting for the lock shows as waiting in `lyx loom status` and sends no quiet notice.
- **PR-Gate fork.**
  The orch stencil narrows it to the run against its board goal and the operator's recorded decisions, and updates its lines on the Publish-verify retry and on Finalize's sync.

Docs and PATTERN land with the code: new `PATTERN-parent-lock`, `PATTERN-told-geometry`, `PATTERN-wait-mark-display-only` and `PATTERN-daemon-wakeups` touches, the `doc.go` files of `parentlock`, `landingshed`, `battenshed`, `shedengine` and `shedverbs`, and `docs/overview.md`.

## Design decisions

- The lock is a record file with a short OS file lock around read-modify-write, not an OS lock held for the hold, since the hold must outlive single step processes.
- Merge-In is its own row rather than part of Publish, so a lock wait, a conflict session and the verify halt belong to one place.
- The seam judges overlap by file and Go package, a superset of the glyphs the board names.
- Darn takes the lock, `Merge-In` and the reroute but has no merge perch; the board scopes the review to loom, and an unlocked darn landing would move a locked parent.
- Out of scope: `lyx pause` itself (`lyx-halt`), detecting a live owner that hangs (`driver-arbiter`), and a lock verb for a hand landing outside a run, which goes through the run's own rows.

## To check by hand

- `internal/fabricengine/parenttips.go` (`FastForwardPair`): its weft step must skip a weft with no upstream, warn past a `PullWeft` failure without stopping the warp side, and record a moved weft HEAD; confirm it matches `Pull`'s weft arm.
- `internal/landingshed/mergein.go` (`syncParent`): after a fast-forward the merged tip must be the tip the pair was moved to, not the pre-fetch upstream tip, so the merge review's range covers every merged commit.
- `config-layers-run-settings` classifies which keys its local layer and run settings may override; `merge_max_bounces`, `merge_auto_accept`, `skip_merge_review` and `parent_lock_idle_release_min` need that classification in whichever task lands second.
- `driver-arbiter` names the Publish verify halt in its eligibility rules and edits batten's notice path; that halt is now Merge-In's, so whichever task lands second updates the other's row name and notice handling.
- Not run: a real suspend or Windows host; the monotonic clock excludes suspend on Linux and macOS, and native Windows is unverified (WSL runs the Linux kernel).
- A live but hung owner holds the lock until the idle bound, batten's backstop or a force release frees it.
