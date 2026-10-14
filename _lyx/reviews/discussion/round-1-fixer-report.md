# Fixer report — discussion, round 1

## Changes

- F1 (BLOCKING), fixed as proposed.
  Premise checked against `internal/fabricengine/remove.go`: after a `GitError`, `removeWarpWorktreeDir` returns the not-a-linked-worktree error once `isRegisteredLinkedWorktree` answers false, so `removePath` is reached only for a still-registered target.
  D7 is rewritten: it states the premise that git deletes the registration before its delete fails, places the bounded retry in that branch, states the ownership proof (git ran, so the gate's registered-linked-worktree and dirtiness checks passed for the target when this `Remove` began, and the target is the path git was asked to remove), recognizes not-empty by git's nonzero exit then the contained removal's not-empty error, and names `syscall.ENOTEMPTY` on Linux and macOS and `windows.ERROR_DIR_NOT_EMPTY` on Windows.
  The re-entry sentence now says a re-entered `Remove` finishes the rest but leaves the leftover as a stray path, matching `inspectPair`, and that the sighting's re-run finished only after the locks were deleted.
  Scope item 7, the D7 acceptance criterion and the plan-writer note for removal follow.
- F2 (MEDIUM), fixed as proposed: D1 and the first acceptance criterion drop the tool event.
  D1's rationale drops "or a tool call" with it.
- F3 (LOW), fixed as proposed: "After D3" is now "After D2".
- F4 (LOW), fixed as proposed: D2 gains a Bound naming the skipped landing check, the turn-end proof and the skill-load timeout (`Spec.SkillLoadTimeout`, else `DefaultSkillLoadTimeout`) measured from the end of `awaitStartup`.
- F5 (LOW), fixed as proposed: D6 resolves the worktree root as each path is resolved, and states a partial-write failure (envelope names the written files and the failing one, nonzero exit).
  The acceptance criteria gain the envelope naming each refused file, a no-file refusal and mode preservation.
- F6 (LOW), fixed as proposed: an acceptance criterion for `gitrepo.PushWithRetry`'s doc comment.

The support log's Review rounds section records the round.

## Disputed

None.

## Deferred

None.
