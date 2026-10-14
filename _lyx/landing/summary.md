# Open the run window as soon as the pair exists; push new pairs detached

`lyx batten run <slug> --window` took about 20 s to show its terminal window.
The window waited for the synchronous code and records pushes of pair creation, the seed push and the inner shed spawn, all of it GitHub round trips the run does not need.

Now the window opens from the Worktree-Create row once the pair is complete and its config reconciled, running `lyx reed attach` in the pair.
Every write to a new pair goes one way: a local commit, then a detached push as `lyx fabric sync` does.

## What changes

- **Early window.**
  The Worktree-Create row opens the terminal through the once-marker Run-Shed already uses, after a fresh create and after an already-complete pair.
  Run-Shed keeps its open, which the marker makes a no-op unless the window was never opened in this run, so a run resumed past Worktree-Create with `--window` still gets one.
  A failed open is warned about and never changes the row's verdict.
  `lyx reed attach` boots the reed session when none exists, and reed serialises boots under its op lock, so the later `lyx loom start --no-attach` joins the same session.
- **`fabricengine` pair creation pushes nothing.**
  `Topology.Add` keeps its pre-flight `ls-remote` probes and the leased deletion of an archived leftover records branch, but no longer pushes the code or records branch.
  With no push inside `Add`, its rollback never sees a pushed branch, so the rollback step that deleted the code branch from origin is removed, as are `pushretry.go`, `pushWeftBranch` and the tests that only covered those paths.
  `AddResult.Pushed` is gone.
- **New `fabricengine.SpawnPairPush`.**
  It spawns the existing detached push for a pair's code and records worktrees from a `*lyxcwd.Location` alone, so callers outside the fabric owner set never name the records worktree, with the gating of `SpawnDetachedPush` (`FABRIC_SKIP_GIT`, `FABRIC_SKIP_PUSH`).
- **`lyx fabric add`** spawns that push after the add and returns without waiting.
  The envelope drops `pushed`, records one `push_spawned` mutation per side, and reports a failed spawn in `push_error` without failing the verb.
- **batten's Worktree-Create and Seed-Child rows** spawn the same detached push instead of waiting on one.
  Seed-Child still commits the seed locally; its push seam is warned about on error and never changes the row's verdict.
- Docs: `fabricengine` and `battenshed` `doc.go`, `lyx batten` and `lyx fabric add` help text, `docs/overview.md`, `PATTERN-fabric-git` and the CLAUDE.md "Watching runs" sentence.

Closes nothing on the issue tracker.

## Design decisions

- **The engine stays local-only; the CLI layers push.**
  A detached spawn from inside `Add` would fork the test binary for every in-process caller.
  `hubforge.AddPairWith` therefore pushes synchronously after `Add` unless its options skip pushing, and `fabriccli` and `battencli` route the spawn through a package seam (`spawnPairPush`) that their tests replace with a synchronous push.
- **A failed or never-run detached push is not an error.**
  Both branches stay committed locally; the pair's next detached push (every fabric commit and sync spawns one) carries them, and `push.autoSetupRemote` creates the upstream on that first push.
  Before, a push failure rolled the add back.
- **Kept synchronous:** the pre-flight `ls-remote` probes, since they decide refusals before the first mutation, and the archived-leftover branch deletion, since a detached push onto the leftover would be rejected.
- `openTerminalOnce` is split so `battenshed.OpenTerminalOnce` is shared by Run-Shed and the Worktree-Create seam.

## To check by hand

- A pair whose first detached push fails stays local-only until its next commit or sync, so another machine cannot adopt it meanwhile.
- A reed config override from the task's run settings, written by Seed-Child after the window opens, can miss the reed session the window boots.
  Run settings target per-strand keys read at spawn, so this is accepted.
- Time `lyx batten run <slug> --window` on a real hub: the Konsole window should appear within about two seconds of the start.
  Tests replace the detached push with a synchronous one, so the real spawn and the timing are not covered.
