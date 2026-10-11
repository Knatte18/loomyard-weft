# Batten lands finished runs itself; the orch wakes only for judgement

## What changes

Batten now runs the landing routine after a child run ends done, and handles one class of halt on its own, so the hub orch is woken only for what needs its judgement.
The orch's per-landing calls (pull, deploy, config reconcile, prune, the teardown check) and its `lyx loom start` loop after an approval go away.

- **Land row.** The batten recipe gains `Land` between Run-Shed and Worktree-Teardown (`internal/battenrecipe`, `internal/battenshed/land.go`, wired in `internal/battencli`).
  It pulls the prime, runs the hub's `deploy` command, snapshots the ready tasks, prunes the board and reads the ready set again.
  It has no `on_stuck`, so a failed step halts batten blocked at `Land` with the pair intact; Worktree-Teardown stays the only terminal.
  An in-flight batten status seeded before this change routes into `Land` after its Run-Shed Done.
- **`deploy` key.** `landing.yaml` gains the hub-wide `deploy` command (`internal/landingshed`), empty by default.
  It is read from the board dir and refused as a `lyx config` write from a pair, the same trust class as `publish_verify`.
- **Notices.** The `done` state-changed notice is removed.
  Land and Worktree-Teardown notify the orch once per failure episode, and once per run when a landing made a task newly ready.
  A landing that succeeds and readies nothing sends nothing.
- **One automatic Publish retry.** A child blocked at Publish by a verify failure is sent to the failure record's target producer and resumed once per batten run, with no notice.
  The second failure gets the blocked notice, which says the retry already ran.
  The Publish failure record now carries `written_at` and the target producer (`internal/verifytree`, `internal/loomcli`).
- **Push retry.** The bounded push retry moves from Add's two branch pushes into `internal/gitrepo`'s plain, never-forced primitives, and also covers a remote `cannot lock ref` refusal.
  The leased update and the remote-branch deletes do not retry.
  An exhausted retry returns a plain push failure, not `ErrPushRejected`, so it never triggers a rebase or reads as divergence.
- **Docs.** The orch role stencil, `CLAUDE.md`'s production exception, `contracts/specs/refusal-spec.md`, and the `doc.go` files of the touched packages follow the new owners.

## Design decisions

- **Land order and locks.** The row holds a hub-level land lock for its whole body and takes the prime lock only around the pull, so a deploy never starves Worktree-Create or Worktree-Teardown.
  The land lock wait is unbounded but honours `lyx batten pause` and cancellation.
- **Prime check.** Pull and deploy are skipped when the child's parent branch is not the prime's checked-out branch.
  Deploy also needs the parent branch in `require_pr_to_base`; otherwise it logs the skip at Info.
  A prime with uncommitted tracked changes is waited on for at most ten minutes, then the row is Stuck naming the paths.
- **Ready snapshot.** The pre-prune snapshot is persisted in a marker, so a restart after the prune still names the right newly ready tasks.
- **Config reconcile.** The row runs none; the deployed binary reconciles at its first start verb through `hubreconcile.Ensure`.
- **Retry ownership.** A retry belongs to a halt only when the record's HEAD matches the worktree and its write time falls inside the halting Publish attempt.
  Anything else is stale and gets the blocked notice.
  A progress marker (`started`, `goto-done`, `resumed`, `capped`) makes the retry restart-safe, and a time cap measured from the first `started` write keeps a stuck driver from silencing the orch.
  This changes the documented rule that batten never spawns or resumes a halted child.
- **Ready notice is temporary.** It departs from the board entry's list of what wakes the orch, until `orch-split`'s run driver starts ready tasks and replaces it.
- **`deploy` trust.** The command runs code from the landed tip and may push outward (the hub's deploy moves `prod`).
  PR-Gate review of landings into `require_pr_to_base` is what keeps task-authored code from reaching it unreviewed; the row does not re-check what PR-Gate did.

## To check or run by hand

- After this lands, the orch sets the hub's `landing.yaml` to `deploy: ./update-plugins.sh` from the prime.
  Until then the `deploy` key is empty and the orch deploys by hand.
- `land-pause-flag-stays-set-on-resume`: confirm that `lyx batten pause` during the land lock wait in `internal/battenshed/land.go` leaves no pause flag set once batten resumes.
- `push-primitive-retry-ordering-untested`: confirm that the retry check runs before `rebaseRetryTriggers` and the `ErrPushRejected` mapping in each primitive of `internal/gitrepo` (`Push`, `PushCoalesced`, `PushRebaseFree`); the ordering is not pinned by a test for every primitive.
- `ready-notice-routed-to-investigating-stop`: check that the orch role stencil sends a ready-task notice to starting the named tasks, not to the Investigating-a-stop path.
- `d5-capped-retry-no-recorded-end`: check that a capped Publish retry in `internal/battenshed/publishretry.go` has a recorded end, so a later halt in the same run is not retried again.
- Land a real task end to end with a non-empty `deploy` and read the row's log, since no run exercised a real deploy.
- Old-binary batten and loom processes still call `hubreconcile.Ensure` after a deploy, a stamp flip-flop `hubreconcile` already accepts; deploys inside running battens make it more frequent.
- A batten killed between Run-Shed's Done and the Land row sends nothing until it runs again.
- The `internal/gitrepo` row in `cmd/lyx/testdata/test-budget.yaml` was raised from 41 to 44.
