```yaml
lens: fix-claims
outcome: reported
```

fix-claims: checking whether the sentences the fix added or rewrote hold against the code at HEAD.

**LOW — card 7 (`07-module-bolt-dir-uses.md`), Intent, second bullet:** "Where one variable feeds both kinds, as in `hubAnchorDeps`, `hubOwnerSeams`, `HubDeps` and `OpenHub`, it splits into a `boltDir` and a `boardDir`…"
- Summary: `HubDeps` has no variable that feeds both kinds, so there is nothing in it to split.
- Evidence: in `internal/tendercli/deps.go`, `HubDeps` calls `fabricengine.BoardDir(location.HubPath)` inline twice, once for the parent-lock store (line 102) and once for `ReadOwners` (line 132). Its one `boardDir` variable (line 151) feeds only `NewBolt` and `boardengine.SpawnSync`, which are both Bolt-checkout uses.
- The claim holds for the other three: `hubAnchorDeps` (owners plus `acquireBranchLock`), `hubOwnerSeams` (stat, pull, owners) and `OpenHub` (`Dirs{Board:}` plus `cfg.Path`).

**LOW — card 7, Intent, fifth bullet:** "Every production `fabricengine.BoardDir(` call outside `internal/fabricengine` not in this list reads or writes the board's own data and stays; `hubforge`'s `Hub.BoardDir`, a test-fixture accessor whose callers reach both, stays, its callers' tests being card 6's."
- Summary: the reason given for leaving `Hub.BoardDir` alone is not accurate, and card 6 does not cover its callers.
- Evidence, non-test callers: `internal/hubforge/seed.go:58-67` writes `configengine.ConfigFile(h.BoardDir(), "fabric")`, the repo-wide config, and commits it. `internal/hubforge/template.go:163` runs `commitRelocatedBinding(tb, hub.BoardDir())`. Both are Bolt-checkout uses, not board data.
- Evidence, card 6: it lists the test members that spell the `"_board"` literal. It does not list the callers of `Hub.BoardDir`.
- Other leftover: `hubgeom/dobbygeom.go:50` fills `dobbyengine.Geometry.BoardDir`, which no production code reads. It is harmless, but it is neither a board-data read nor listed.

Every other claim I checked holds at HEAD:
- **Card 2 (`allowedImports`):** the `leaf_enforcement_test.go` header and the `allowedImports` comment both enumerate the import set.
- **Card 8 (`//lym:guard`):** `TestEnforcement_GeometryLiterals` carries `//lym:guard` directly above its func line (`internal/lymcwd/enforcement_test.go:202-203`).
- **Card 9 (Reconcile row):**
  - `repairPairWiring` re-wires only when `checkJunctionHealth` is unhealthy.
  - A removed link yields "warp %s junction missing" (`reconcile.go:722-725`), so the reworked row is reachable.
- **Card 9 (load order):** loading config before `seedLymJunction` matches the junction-name precedent.
- **Card 10 (`FastForwardPair`):**
  - It holds `localHEAD` and `upstreamSHA` before `ResetHard` and returns `NewHead: upstreamSHA`.
  - `landingshed/finalize.go:379` prints nothing from `FastForwardResult` beyond its refusal text, so "the warning is logged only" holds.
  - Fabric has `f.warpPath`.
- **Card 13:**
  - `restoreReplayTip`'s message reads "after a failed board replay" (`destroy.go:1985`).
  - The six function names and files named for the widened guard match: `bolt.go:335`, `pushclassified.go:521`, `pull.go:505`, `originreconcile.go:823`, `originreconcilemoves.go:510` and `:582`.
- **Card 6:** `yamlengine` and `vscode` are in `go list -deps ./internal/fabricengine`, and none of the other in-package test packages listed are.
- **Card 19 (bound paragraph):** the annotation is help text only (`clihelp/annotations.go`), and the adoption and compaction exceptions match D4.
- **Not verified:** whether `lymcwd.Resolve` succeeds from the Bolt checkout, which the new card 19 row needs before `RequireDrivableWorktree` can refuse. No test outside fabric reaches that refusal, so there is no evidence either way.

Tool calls beyond context:
1. Grepped the `dobbyengine`/`dobbycli` uses of `BoardDir`, the non-test callers of `Hub.BoardDir`, the `enforcement_test.go` guard lines and `finalize.go:375-395`, to check the card 7 exemptions, card 8's guard marker and card 10's "not printed" claim.
2. Grepped `checkJunctionHealth`, `RequireWarpWorktree` and `tendercli` `HubDeps` lines, to check card 9's re-wire trigger, card 19's refusal row and card 7's split claim.
3. Grepped tests outside fabric for the Bolt-checkout refusal, to check whether card 19's new row is reachable.
