# Plan-Review round 1 — fixer report

Every finding was checked against the code and the decision record before fixing.
`lyx loom validate-plan` passes over the fixed plan, with only informational `create-new-unit` rows.

## Fixed

- **F1** (BLOCKING): confirmed in `cmd/lyx/stencilseed.go`.
  `seedStencilsAt` runs `seedSubtree` once per subtree, so a per-subtree pull would drop the stencils seed commit during the specs pass.
  Card 7's `Bolt.PullThenCommitWritten` now takes an ordered `[]BoltWrite`, each with its own `Message` and `Write`.
  It takes the lock once and pulls once, then commits each write separately, and `BoltWriteResult` carries `SHAs`.
  Card 8's `seedSubtree` is re-signed to return the subtree's `BoltWrite` and whether a write is due.
  `seedStencilsAt` makes one call carrying both writes.
  The scenario test's first step now has both subtrees due and asserts that both seed commits survive.
  Cards 10 and 11 pass one `BoltWrite` and list `BoltWrite` under `Uses`.
- **F2** (BLOCKING): the new Bolt tests move to a new `package fabricengine_test` file, `internal/fabricengine/boltpullfirst_integration_test.go`, on card 7's `Create:`.
  They run over a `hubforge` hub whose upstream a second clone of `RecordsBare` moves.
  `bolt_integration_test.go` leaves card 7's targets.
- **F3** (MEDIUM): confirmed locally: `tmux -L lyxprobe-x list-sessions` printed `error connecting to /tmp/tmux-1000/lyxprobe-x (No such file or directory)`.
  Card 12's `IsNoServer` now admits both the `no server running on` form and the absent-socket form.
  `TestListSessions` gains a row for each form and a row for an unrelated failure.
- **F4** (MEDIUM): confirmed that `isAnyWorktreeOf(target, target)` always holds.
  `ownedBoardWorktree` now requires the `BoardDirName` base name and that a weft repo directly in the target's parent (`WeftWarpSlug`) registers the target as a linked worktree (`isRegisteredLinkedWorktreeIn`).
  The Intent states the bound: only a hub's own `_board`, never with force.
  A new `TestBoardDropRequest_RefusesForeignBoard` covers the refusal.
- **F5** (MEDIUM): `dirtyTrackedExcept` has the inverse sense, refusing outside its paths, so card 7 adds a `dirtyTrackedOn(paths)` dirtiness kind in `checkPathDirtiness`.
  The new kind refuses only on a dirty tracked path that the ahead commits touch; `seedCommitsAhead` now returns those paths.
  `ResetKeep` already refuses with nothing changed on the remaining collision, a dirty path that upstream moved.
  A dirtiness refusal, or a reset or fast-forward that refuses over a local change, returns a new `BoltSkipDirty`.
  Its Warn names the dirty paths, the next `lyx board sync` and a re-run.
  Two subtests cover dirt on an untouched path being carried across the drop and dirt on a touched path skipping.
  The overview's `skip-warns-once-in-bolt` decision and cards 10 and 11 name the dirty skip.
- **F6** (MEDIUM): decided in card 13.
  The push retry passes a local `Mutations` rooted at the hub's container and logs the dropped SHAs from it at Info.
  `Bolt.Push(opts) error` has no result type, and its callers' push paths are out of scope per the record's `Out:` list.
  The primitive still appends after an observed change, and the pre-run's logged, not enveloped, seed record is the precedent.
- **F7** (LOW): card 7's integration test keeps only one refuse row, the labelled seed message touching a path outside the subtrees.
  That row exercises `CommitDetail`'s paths, and card 6's `TestSeedCommitSignature` owns the rest.
- **F8** (LOW): the push retry is split out of card 7 into a new card 13, `bolt-push-seed-drop-retry`, with its own test and `doc.go` sentence.
  The `internal/fabricengine` budget is split so card 7 raises it by two and card 13 by one.
  Card 13 sits last rather than eighth because no earlier card depends on it.
  Inserting it after card 7 would rename every later card file, and this round may change files only through Edit and Write.
  See Disputed for the drop primitive.
- **F9** (LOW): `internal/fabricengine/livestate_mutationoracle_test.go` is on card 7's `Edit:`, and `KindCommitsDropped` is classified there as a git-state kind (`false`).
- **F10** (NIT): `internal/fabricengine#Bolt` is dropped from card 7's targets.
- **F11** (NIT): card 3's `ImpactSummary` now says writer keys are written only from card 4 on.

Collateral edits:
- Card 5's `ImpactSummary` now says that card 13 is the sentinel's first caller.
- The Card Index rewrites line 7 and adds line 13.

## Disputed

- **F8**, in part: the finding asks for the drop primitive in a card of its own, ahead of the pull-first verb.
  The rubric's granularity rule says "a private supporting type, or a constructor inseparable from its type, belongs in the other symbol's card", and gives its own card only to "a symbol that is part of a module's public surface, or reused across packages".
  `dropSeedCommits`, its ownership and dirtiness kinds and `boardDropRequest` are all unexported and used only inside `fabricengine`.
  They therefore stay on card 7 with `pullFirst`, their first consumer, whose slug already names the drop.
  The independently reviewable public surface the finding names, the `Bolt.Push` retry, is split out as described under Fixed.

## Deferred

None.
