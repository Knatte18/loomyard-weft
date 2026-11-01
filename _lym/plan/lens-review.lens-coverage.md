```yaml
lens: coverage
outcome: reported
```

Coverage lens: I checked each decision, acceptance criterion and card-introduced branch against the planned test rows. Defects only.

1. **BLOCKING — D9, cards 4 and 7: card 7's guard will fail on `_board` hits that card 4 neither lists nor exempts.** Card 7's `TestEnforcement_BoltDirNameDeclaredOnce` scans every `.go` file, test files and comments included, and exempts only `anchor.go`, `enforcement_test.go` and `livestate_doc_test.go`. These hits sit in members card 4 does not list:
   - `internal/fabricengine/livestate_manifest_test.go:28` (a comment in no member, so the file's members need listing)
   - `internal/fabricengine/export_test.go:321` (a comment in the declarations after `SeedFabricConfigForTest`)
   - `internal/fabricengine/livestate_verbs_test.go:616` (the region ending `pruneCase`, before `cleanupCase`)
   - `internal/fabricengine/remove_guard_integration_test.go:53` (`TestRemove_RefusesPrimeWorktreeAndLeavesItIntact`, or the comment after it)

   Card 4 lists `cleanupCase` but none of these, so card 7's card gate fails as planned. The method of hand-listing members misses hits that sit in comments outside members. Card 6's list may have the same gap: I did not check `internal/fabriccli/cli_test.go` or `internal/configcli/configcli_test.go`, which each hold one hit.
2. **MEDIUM — D6, card 9: the rewrite path of `Fabric.reanchorAfterRewrite` fires "took in" but no row reaches it.** Card 9's Intent fires after each of its `ResetHard` calls. `TestWorktreeSeam_Events` only drives a fast-forward pull; no row covers a non-fast-forward pull (with or without index entries).
3. **MEDIUM — D6 AC "a records or Bolt fast-forward fires nothing", card 9: unchecked in `ReconcileOrigin`.** `ffWorktree` is shared by `reconcileRecordsBranch` (`originreconcilemoves.go:197`) and the code side (`:392`). The test asserts "nothing" only for `Bolt.Pull` and a weft-only fast-forward. No row checks that a `ReconcileOrigin` records or Bolt fast-forward records no event, and no row checks the paths a successful code-branch move passes.
4. **MEDIUM — D5, card 8: `lym fabric reconcile` re-wiring is not tested for seeding the lines.** D5 names reconcile as a seeding site; card 8 reaches it through `wireJunctionSet` (`reconcile.go:461`). `TestSeedWarpExclude_SeedsWarpOnlyOnce` covers only `CopyHub` + add + `WireJunctions`. `AddLocal` likewise has no row.
5. **MEDIUM — D8 and card 16: three branches of `heddleengine#Replayer.Apply` have no row in `TestReplayer`.**
   - An entry origin no longer holds is reported lost.
   - A removal of an entry origin changed is reported lost (the `!Same` branch).
   - An added slug origin also added is reported lost through `AddClash`.
6. **MEDIUM — D7 AC "no `board.json`, overview or design file name, board status `done` or board-named replay type left in fabric's replay, ended-set or owner-read code", card 12: no test checks it.** Card 12 carries it only as Delete targets. No scan or row would fail if a later edit put such a name back, unlike card 7's guard for `_board`.
7. **LOW — D7 AC "a claimed path with no replayer fails naming the registration", card 12 and Shared Decisions: the plan reads it differently from the record.** The plan cherry-picks any path no registered replayer claims and fails only when no run ledger is registered. A `heddle.json` commit in a process that registered only the board's replayer is therefore cherry-picked, not refused. The Shared Decision states this but does not mark it as a departure from the criterion.
8. **LOW — D3, card 14: two branches have no row.**
   - The `.heddle` directory itself off `Excluded` (the refusal covers "its file, or `.heddle`"); only a target file off the list has a row.
   - A `CLAUDE.md` only in `.claude` giving no import (the record names "there or in its `.claude` directory").
9. **LOW — D2, card 18: two refusals and one Intent branch have no row.**
   - `show` with an unknown target.
   - A hub with no Bolt checkout.
   - `render` printing render warnings.
10. **LOW — card 8: a config load failure inside `wireJunctionSet` (a new failure source for every wiring caller) has no row.** Neither does card 9's `EventFor` failure being turned into a warning.
11. **LOW — D6, card 9: with the seam installed, `Topology.add`'s "made" warning (a seam that fails during an add) has no row.**

Tool calls:
- grep of `_board` in 14 test files: my context did not show which members of the fabric test files hold the hits.
- awk lookup of the declarations enclosing 8 of those hits: to check them against card 4's member list.
