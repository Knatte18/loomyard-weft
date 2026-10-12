# Merge review perch and a parent merge lock that survives sleep and crashes

`merge-review` · Task · enhancement, loom, burler, batten, shed · running · Discussion-Write

## Seen (2026-10-11)

At PR-Gate the orch forks a reviewer over the whole diff, but Webster-Burler has already reviewed the task's code.
What no review has seen is the interaction with what landed on the parent while the task ran: Publish merges the parent in, and only verify runs over the result.
Verify catches what `go test` and PATTERN's scans enforce; invariants enforced only by review, and the semantic interplay of the task's code with the parent's new code, go unchecked.
Millhouse's `/mill-merge` (`plugins/mill/skills/mill-merge/SKILL.md` and `mill-merge-in/SKILL.md` in the Millhouse repo) has the same gap: under a merge lock it resolves conflicts once, replays verify with a verify-fix agent, and squashes, with no review of the merge.

## Design direction (operator and orch, 2026-10-12)

Everything here works for any parent branch, never for a fixed one: `main` is only the parent of most hub runs today.

### Merge review

- An optional perch after Publish's merge, before PR-Gate, reusing the perch machinery: its own round budget in config, set to 1 with auto-accept on cap by default, so one review and one fix, then on.
  A finding the fix cannot settle escalates like any perch.
- The seeder decides whether the round is needed: it runs only when the commits the merge brought in from the parent touch the task's packages or glyphs (quarry can answer this), and is skipped otherwise.
- The reviewer is cold and reads only that overlap, against `PATTERN.md` and the task's code.
- With the lock below held, the PR-Gate fork narrows to what needs the orch's context: the run against its board goal and the operator's decisions.

Measured on the ten most recent landed tasks (`.scratch/webster-round-times.md` in the hub prime, 2026-10-12): a Webster review round took a median of about 2 minutes (1-8), Publish about 3 (2-4), the PR-Gate fork about 1.25 (1-2), with 1-4 rounds per task.

### Parent merge lock

- Like Millhouse's merge lock: from the moment Publish starts merging the parent in, through conflict resolution, verify, the merge review and PR-Gate, to Land, no other run merges into or lands on that parent; another run reaching Publish on the same parent waits.
  So the parent cannot move under the review, and no merge or conflict resolution is redone.
  Landing today takes no such lock: only the push or PR merge itself is atomic.
- One lock per parent branch, visible to every worktree on the hub.
- Validity follows the owner process, never wall-clock time, because the host may hibernate at any moment (see note `lyx-halt`): the lock records the owner's pid and start time, as `loop.pid` does, and is stale only when that process is dead.
  A hibernated host keeps the lock valid however long it sleeps; a crashed owner frees it at once for the next run.
- The lock is held only while the run makes progress on its own:
  - It is released at once on any stop: `awaiting` (escalation, parent review, PR-Gate waiting on a human), `halted`, `stuck` or pause.
  - The owning shed loop reads shuttle's session state and releases the lock when its agent sits in a waiting state (`held`, `asking`, `idle-stalled`) past a bound measured on the process's monotonic clock, which on Linux does not advance during suspend, so sleep never counts as waiting.
  - Batten, which already sends agents-idle notices, releases a lock its owner did not as a backstop.
  - A verb lets the orch or the operator force a lock free; `lyx pause` from `lyx-halt` releases every lock on purpose before a sleep.
- A run that lost the lock and finds the parent moved merges the parent in again when it resumes; the seeder then judges only the new commits for the merge review.
- A process that hangs while alive is the driver's to detect (`driver-arbiter`).

## Merged: `prgate-stale-base` (2026-10-11, bundled 2026-10-12)

`burler-split-budget` landed while `orch-wake-policy` waited at PR-Gate with PR #564.
GitHub then reported #564 `CONFLICTING` against the moved parent: 16 files changed by both tasks.
The PR-Gate review fork would not approve a conflicting PR, so the orch sent the run back with `lyx loom goto --to Publish` and `lyx loom resume`, where Publish merges the parent and its conflict session resolves the conflicts.
The operator's idea: when the parent has moved past the PR's base while a run awaits at PR-Gate, loom sends the run back to Publish by itself.
With the lock this arises only when a run lost the lock or the parent moved outside lyx (a hand push); the task builds this send-back as the path for both.

## Open

- Where the lock lives so every worktree on the hub sees it, and how batten's Land row and a hand landing by the orch take and release it.
- The waiting bound, and whether it lives in config.
- Whether a run waiting for the lock shows as waiting in `lyx loom status` and batten notices, so the orch does not read it as stuck.
