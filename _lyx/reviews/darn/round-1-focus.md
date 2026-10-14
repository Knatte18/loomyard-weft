---
round: 1
focus:
  - "Scope: the board entry asks only for the early window and says the pushes stay as they are; the summary says the parent widened it to detached pair pushes and a detached seed push. Check `_lyx/discussion/decision-record.md` for an `Added after Discussion` entry licensing each of the three changes, and whether the diff makes any change beyond them."
  - "`internal/fabricengine/add.go`, `remoteleftover.go`, `weftwiring.go`, `destroy.go`: with `Add` no longer pushing, does any rollback or adoption path still assume a branch pushed by this call? Does the leased deletion of an archived leftover weft branch as the last step still leave consistent state when a later step fails?"
  - "Callers of `Topology.Add` (`lyx fabric add`, batten Worktree-Create, `hubforge.AddPairWith`, any other): does each spawn the detached pair push through `fabricengine.SpawnPairPush`, and does a spawn failure stay non-fatal with the `push_spawned` / `push_error` envelope the summary claims?"
  - "`internal/battenshed/innerrun.go` and `internal/battencli/wire.go`: is the once-marker in `OpenTerminalOnce` correct for the Worktree-Create open, the Run-Shed open after spawn and a resumed run? Is the window opened only after `Add` and the config reconcile succeed, and only with `--open-terminal`?"
  - "Seed-Child `PushSeed`: does the detached push replace the synchronous records push without changing `CommitSeed`, and is the push-child failure logging (both worktree paths) in the bypass handler present and correct?"
  - "Test economy: for the deleted push-retry and rollback tests, is any still-reachable behavior left uncovered? For new or rewritten tests (`refuseRecordsCommits`, `pushNewPair`, the `TestMain` seam replacements), does any duplicate an existing test? Is the new `SpawnPairPush` / `OpenTerminalOnce` behavior covered?"
  - "Docs and claims: `doc.go` files, `pattern/PATTERN-fabric-git.md`, `docs/overview.md`, CLAUDE.md and the help text must match the new behavior; check each claim in `_lyx/landing/summary.md` against the diff, including that `AddResult.Pushed` is gone with no leftover reader and that the test-budget file is edited if a package passes its row."
---

Focus is on scope, since the entry's stated scope is narrower than the diff, and on the dead-code removal in `fabricengine`, where a leftover assumption would be a correctness bug the verify gate does not see.
