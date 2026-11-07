```yaml
lens: interactions
outcome: reported
```

Task: interactions lens. I paired up the decisions that act on the same state, file, key or ordering, then checked the acceptance criteria and constraints against each decision they touch.

**Pairs checked**
- D2 × D3, failed-build mark against the poisoned flag: held. A registered copy checks failed, a shape copy checks poisoned.
- D2 × D3, D2's rule that the board's HEAD equals its upstream after a build, against D3's amend and push of the tip: held. The criterion reads "history as built", and the reflog survives the git fold.
- D2 × D3, the order build, build function, pack, fingerprint: held for shapes and registered templates alike.
- D2 × D4, builds running through the initialisers D4 rewrites: held.
- D2 × D7, D7's "moved as they are" against D2's refusal of `Cleanup`, `Setenv` and `TempDir`: held. D7 keeps steps with outside paths in the test.
- D2 × D7, the board-state guard against D7's exemption for steps a build cannot run: **broken in wording** (finding 3).
- D3 × D6, the three `TestAnchorChange` refusal rows on one `CopyHub` against D3's bound that the split adds no `SharedHub` site: held.
- D1 × D3, a packed template meaning a copy's refs sit only in `packed-refs`: held. D1 falls back to `packed-refs`, and a criterion covers it.
- D1 × D3, the `gitdir` files rewritten as text on relocation: held.
- D5 × D6 × D9, `replay_integration_test.go` and its `EndedSet` rows in task 5 only: held. No D7 fixture lives in that file.
- D6 × D7 × D9, `livestate_matrix_test.go` (`runCell`), `anchorchange_integration_test.go` and `originreconcilemoves_integration_test.go` (`endedHub`): held. Tasks 7 and 8 wait for task 6.
- D5 × D7 × D9, the `internal/hubreconcile` `NewHub` sites in task 9: held.
- D7 × D8 × D9 for `internal/hubreconcile`: held. `TestEnsure_HeldLockTimesOutWithoutStamping` and `newStaleHub` share `ensure_integration_test.go`.
- D7 × D8 × D9 for `internal/battencli`: **broken** (finding 2).
- D4 against its own acceptance criterion: **broken** (finding 1).
- D10 × D9, the re-measure after every task has landed: held.
- The Constraints against D1, D2 and D8: held.

**Findings**

1. MEDIUM, consistency, D4 and its D4 acceptance criterion.
   - The criterion has a repository from each initialiser read "the five keys at today's values".
   - The bare initialisers write only three of them today: `core.fsmonitor`, `maintenance.auto` and `gc.auto`, with no `user.name` or `user.email`.
   - So the criterion can only pass by adding keys, which breaks D4's "the keys and values stay as today".
   - The decision's own list of "five keys" for every initialiser carries the same misreading.
   - Evidence: `internal/gitkit/gitkit.go:149-157` (`initBareRemote`), `internal/hubforge/hub.go:467-477` (`initBareRepo`), against `gitkit.go:131-138` and `hub.go:457-465`.

2. MEDIUM, consistency, D7's `internal/battencli` bullet × D8 × D9 task 11.
   - D7 names the fixture `beamHub`, which is defined and used only in `beamguard_integration_test.go`.
   - The start state D7 describes, "a beam, a member and a plain pair", is `noticeHub` in `notices_integration_test.go` (beam `b`, member `m`, plain `p`).
   - D8 puts `TestNoticeSite` (`notices_integration_test.go:42`) in the battencli task and says "No task opens a file only to move a test".
   - If the task moves the fixture D7 names, it never edits `notices_integration_test.go`, and D8's own rule forbids that test's move.
   - If it moves `noticeHub`, D7 names the wrong fixture.
   - Evidence: `internal/battencli/beamguard_integration_test.go:26`, and `internal/battencli/notices_integration_test.go:21-29,42`.

3. LOW, consistency, D2's board-state rule × D7's exemption clause.
   - D2 fails any build that does not leave the board's HEAD equal to its upstream tip.
   - D7 keeps only a step "that leaves the board behind its upstream" in the test.
   - The usual unequal case is a board ahead of its upstream: a Bolt commit that is never pushed, as `SeedFabricConfig` leaves it (`Bolt.Commit` "never pushes").
   - D7 neither keeps such a step in the test nor admits it to a build, so a mover following D7 gets a build D2 fails.
   - Evidence: `internal/fabricengine/bolt.go:49`, `internal/hubforge/seed.go:72-74`, and the `SeedFabricConfig` calls in `internal/hubreconcile/ensure_integration_test.go:520,991-1012`.

**Tool calls**
- grep for where each named fixture and test is defined: the manifest had no file locations for D6, D7 and D8's symbols.
- grep for the fixture callers per package and a read of `newBadReedUpFixture`: needed to map files to tasks 5 to 11, which the manifest did not cover.
- read of `SeedConfig`, `SeedFabricConfig` and `Bolt.Commit`'s push behaviour: needed to judge D2's board rule against D7's builders.
- grep for the seed calls in moved fixtures and for `notices_integration_test.go`'s fixture: needed to judge D8's battencli file claim.
