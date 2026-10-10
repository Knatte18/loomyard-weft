# Fixer report — plan, round 4

## Fixed

### F1 — the seed-commit drop is not serialised against a concurrent pusher's `pull --rebase`

Validated against the code: `gitrepo.PushCoalesced` (`internal/gitrepo/push.go:192`) holds `.gitrepo-push.lock` (`gitrepo.PushLockFileName`) across `pushWithRebaseRetry` and never takes `board.lock`.
`boardengine.Sync`'s step pushes after `commitDirty` released `board.lock`, and the other `Bolt.Push` callers push outside any board lock, so the board write lock alone does not exclude another process's rebase in `_board`.

- `_lyx/plan/07-bolt-pull-first-seed-drop.md`: `pullFirst` holds `gitrepo.PushLockFileName` in the board dir for its whole run (fetch, drop, fast-forward), taken after the board write lock and released before the writes; the Intent states why and why the order cannot deadlock.
  `Uses:` gains `internal/gitrepo#PushLockFileName`; the planned `doc.go` text names the lock order; `TestBolt_PullThenCommitWritten` gains a subtest in which the test holds the push lock, Bolt's HEAD stays at the original tip, and the drop and write land only after release.
- `_lyx/plan/13-bolt-push-seed-drop-retry.md`: `PushRecorded` takes the push lock after the board write lock, once `pushWeftAt` has returned, holds both across the ahead listing and the drop, and releases both before the retry; `Uses:` gains `internal/gitrepo#PushLockFileName`, and the planned `doc.go` text puts the retry under the same lock order.
  No separate lock test for this site: a push lock held by a test blocks `pushWeftAt` before the drop is reached, so it cannot isolate the drop's wait; card 7's subtest pins the lock rule.

`lyx loom validate-plan` reports `ok: true` after the edits, with only the informational `create-new-unit` notes it reported before.

## Disputed

None.

## Deferred

None.
