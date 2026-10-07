# Push the task branch with the run records, and retry benign rejections

A task pair's code branch now reaches the remote after every loom transition, whatever made its commits.
A status push that the remote rejects without a real divergence recovers by itself.
Before this, loom pushed only the records side, so an agent's raw `git commit` stayed local, and a status push racing a detached `lyx fabric push` child on the same records branch was reported as rejected.

Closes #418

## What changes

- `fabricengine.PushPairAnchored` pushes the code side, then the records side, each rebase-free, under the absorbing `fabric.push.lock`.
  It attempts the records side even when the code side failed.
  Each failure names its side and worktree path and wraps the side's error, so `errors.Is(err, gitrepo.ErrPushRejected)` holds for one rejected side and for both; two failures come back as an `errors.Join`.
  `fabricengine.IsPushRejected` is the side-neutral predicate for callers that may not name a side.
- Loom's per-transition status seam and `lyx loom commit-records` push through it.
  The seam still warns on every push error and continues.
- `lyx fabric push` pushes both sides with an unbounded lock wait, so it waits out the detached child a landing commit just spawned.
  A rejection that stays is now an error envelope, not a swallowed warning.
  `lyx fabric sync` hands both worktree paths to its detached child.
- `PushAnchored` (records side only, used by batten's seam in the prime) takes the same lock and gains the retry.
- A rejected push fetches that side's upstream and pushes once more when the remote tip is an ancestor of, or equal to, local HEAD.
  A real divergence is left alone and warned about; nothing is rebased, merged or forced.
  The retry lives in the anchored pushes, not in `gitrepo.PushRebaseFree`.
- Status-side lock acquisition waits a bounded, injectable time (`StatusPushLockWait`) through the new `lock.AcquireWriteLockWithin`, and on expiry returns `ErrPushLockBusy`, which the seam warns about.
  `gitrepo.Repo.HasUnpulled` was added alongside.
- The `commit-records` push-failure refusal splits into a rejected row and a failed row, in the code and in `contracts/specs/refusal-spec.md`, both transient.
  The envelope no longer claims the records were not pushed, since only the code side may have failed.
- `PATTERN-push-both-sides` records the invariant; the `fabricengine`, `gitrepo` and `docs/overview.md` text and the `fabric push`/`sync` long help are updated.

## Decisions to review

- The bound on the lock wait covers only a waiter.
  A holder's length depends on the network, since `gitexec` sets no command timeout, so a hung status push now also blocks detached children behind it until git returns.
- `lyx fabric push` and `lyx fabric sync` push unconditionally, so in the prime they publish unpushed commits of the parent branch even when the call committed nothing.
  The push is plain and never forced; a behind or protected parent branch yields an error envelope after the records side was pushed.
  Batten's seam is unchanged and still pushes the records side only.
- `lyx fabric push` no longer runs `PushCoalesced`'s `pull --rebase` on a rejection; the single fetch-and-retry applies instead.
- `lyx fabric sync` records one `push_spawned` entry per side it hands to the child.
- The cause of #418 is inferred from the code, not from the drive report.

## To check by hand

- The new push tests that need a real remote are `integration`-tagged and were not run by hand here; run them with the integration tag.
- Agents that amend or rebase an already-pushed commit make the code side diverge, so the push warns every transition until the branch is resolved.
