```yaml
lens: guard-reach
outcome: reported
```

Lens: guard-reach. For each guard, refusal or test row a card adds, I checked whether a real entry point can reach it and whether the guarded site recurs unguarded.

**Per guard, reachable or not**
- Card 7 `TestEnforcement_BoltDirNameDeclaredOnce`, including its matcher self-check and its one-declaration count in `anchor.go`: reachable. Its exemption of `livestate_doc_test.go` is a fidelity question, not a reach one.
- Card 8, config-load failure in `wireJunctionSet`: reachable from a direct `WireJunctions` call. In production, `RepoWiredNames`/`WiredNames` fails first, but the card's own row bypasses that.
  - The row's "seeds nothing" assertion holds only if the config loads before the junction steps. See finding 4.
- Card 8, `TestSeedWarpExclude` Reconcile row ("lines stripped, `Topology.Reconcile` seeds them again"): **not reachable.** See finding 1.
- Card 8, `AddLocal` row and the twice-run idempotence row: reachable.
- Card 9, `fireWorktreeEvent` with no seam installed: reachable.
- Card 9, failing-seam rows on add, pull and `ReconcileOrigin`: reachable.
- Card 9, `EventFor` failure on an unparseable config: reachable. `Fabric.Pull` loads no fabric config before its warp arm (no `LoadConfig` or `WiredNames` in `pull.go`).
- Card 9, rewrite row through `reanchorAfterRewrite`: reachable through the `buildReconcileFixture` prime path.
- Card 9, `originMover.move` code-side row: reachable. Locks are released between moves, and no `ReconcileOrigin` caller holds the Bolt write lock.
- Card 9, "took in" fire sites: the guarded site recurs unguarded at `Fabric.FastForwardPair`. See finding 2.
- Card 12, `ErrReplayerMissing` on a claimed commit with only `fakeReplayer` registered: reachable.
- Card 12, `ErrReplayerMissing` on the ended-set read: reachable.
- Card 12, `ErrReplayerMissing` on the owner read: reachable. `readOriginOwners` runs before `readOriginEnded` (`originreconcile.go`, `ReconcileOrigin` body).
- Card 12, cherry-pick with nothing registered: reachable.
- Card 12, `set-aside/replayer/fake/…` tag: reachable.
- Card 12, empty-entry ledger fallback: reachable only through the `emptyEntryLedger` fake. The card says so itself.
- Card 12, `TestBoltReplay_NamesNoBoardFile`: the guarded property recurs in files the guard does not scan. See finding 3.
- Card 13, all `ValidateEntry`, `Set`, `Remove` and `Decode` refusals: reachable from tier0 rows.
- Card 14, `.heddle` file or foreign-file warning: reachable.
- Card 14, off-`Excluded` refusal: reachable directly, and in production by narrowing `warp_exclude`.
- Card 14, Bolt-lock wait timeout and render lock: reachable through the injected `BoltLockWait`.
- Card 14, unmarked-file skip and stale marked removal: reachable.
- Card 15, `TookIn` with no target selected: reachable.
- Card 16, lost, refused and `Empty` branches: reachable from tier0 rows.
- Card 17, empty `Board`: reachable. `ConfigDirs` leaves `Board` empty outside a hub.
- Card 17, no reader installed and reader error: reachable.
- Card 18, unresolvable cwd: reachable from a temp directory.
- Card 18, missing Bolt checkout: reachable. The prime resolves from its own marker or falls back to "." (`lymcwd/anchor.go:162-186`), so `Resolve` does not refuse first.
- Card 18, `namedWorktree` unlisted or prunable: reachable.
- Card 18, render lock timeout and `spawnSync` failure: reachable.
- Card 18, `RequireDrivableWorktree` refusal (Bolt checkout, records sibling): reachable in principle, but no test is named. See finding 5.
- Card 19, pre-run pin: reachable through the AST parse.

**Findings**

1. **MEDIUM**, design. Card 8 (`08-fabric-exclude-list.md`, Tests, "with the three lines stripped from `info/exclude`, `Topology.Reconcile` of the pair seeds them again"), with D5's "`lym fabric reconcile`" seeding.
   - **Summary:** Reconcile reaches `wireJunctionSet` only for a pair whose junctions are unhealthy. A healthy pair with stripped exclude lines is never re-seeded, so the row fails as written and D5's reconcile-seeding promise does not hold for healthy pairs.
   - **Evidence:** `internal/fabricengine/reconcile.go:448-466`. `repairPairWiring` calls `wireJunctionSet` only inside `if !junctionHealthy`.
   - **Fix:** seed the exclude list on reconcile's healthy branch too, or change the row so it breaks a junction first.

2. **MEDIUM**, consistency. Card 9 (`09-fabric-worktree-seam.md`, "Where it fires").
   - **Summary:** `Fabric.FastForwardPair` fetches origin and moves a checked-out code branch to its upstream with `ResetHard`, the same move D6 says fires "took in" "wherever fabric moves a checked-out code branch to origin's commits". Card 9 neither fires the event there nor says why D6's landing exception covers it.
   - **Evidence:** `internal/fabricengine/parenttips.go:248-283` (`f.code.Fetch()` then `f.ResetHard(rec, upstreamSHA)`). Callers: `internal/landingshed/finalize.go:379` and `finalize_pr.go:247`.

3. **LOW**, design. Card 12 (`12-bolt-replayer-registry.md`, `TestBoltReplay_NamesNoBoardFile`).
   - **Summary:** The guard parses only `replay.go` and `boltreplay.go`. The ended-set and owner-read code that D7's acceptance covers ("no `board.json` … left in fabric's replay, ended-set or owner-read code") also lives in files it does not scan, so a board file name could return there unguarded.
   - **Evidence:** those files are `pull.go:536-558` (`weftEndedByOrigin`), `pushclassified.go:540-562` (`goneEntryEnded`), `originreconcilemoves.go:583-597` (`addLocalCreatedStamps`), `originreconcile.go:819-830` (`readOriginOwners`) and `bolt.go:333-340` (`endedByOrigin`).

4. **LOW**, consistency. Card 8 (Intent: "seeds `WarpExcludePaths()` … after the junction lines"; Tests: "`WireJunctions` fails naming the config load and seeds nothing").
   - **Summary:** The card does not say the config loads before `seedLymJunction`. If it loads where the seeding sits, after the junction steps, the links and junction exclude lines are already written when the load fails, and "seeds nothing" cannot hold.
   - **Evidence:** `internal/fabricengine/junction.go:115-139`. The order is `seedLymJunction`, then `seedWeftArtifactExcludes`, then `seedGitExclude`.

5. **LOW**, consistency. Card 18 (`18-heddle-cli.md`, pre-run paragraph: "a cwd `RequireDrivableWorktree` refuses (the Bolt checkout, a records sibling) keeps that function's own text").
   - **Summary:** The card prescribes this refusal branch, but no `TestHeddleCLI` row runs a verb from the Bolt checkout or a records sibling.
   - **Evidence:** the rows in card 18. `fabric.go:135-162` holds the refusal text.
   - **Also:** on a subpath-anchored hub, `lymcwd.Resolve` may refuse first with `ErrCwdOutsideAnchor`. The handler's own `lym board get` from the weft checkout hit exactly that error earlier in this session.

Tool calls:
1. `reconcile.go:425-470`, to see when Reconcile reaches `wireJunctionSet`. The context held only the call site line.
2. A grep of `add.go` for locks, and of the pull files for config loads, to check that the seam fires with no Bolt lock held and that the `EventFor`-failure row is reachable. The context lacked both.
3. `lymcwd/anchor.go:140-200`, to check whether `Resolve` refuses first when the Bolt checkout is missing. The context held only the `boardDirName` lines.
4. A grep for `ReconcileOrigin` callers, to check whether any caller holds the Bolt write lock while the code-side event fires. The context lacked the callers.
5. A grep for `CommitWritten`/`PullThenCommitWritten` callers, to rule out a worktree add or code move inside a Bolt-locked closure. The context lacked them.
