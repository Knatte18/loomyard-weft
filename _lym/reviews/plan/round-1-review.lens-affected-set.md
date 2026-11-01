```yaml
lens: affected-set
outcome: reported
```

Task: affected-set lens. For each change a card makes, I traced what reads or asserts it and checked which cards cover that set.

**Per change: the affected set, and the cards that cover it**

1. **`runproc` gains an `internal/lymcwd` import (card 2).**
   - Set: `internal/runproc/record.go` (`HubOwner`), `internal/runproc/doc.go`, the `PATTERN-leaf-packages` background, and `internal/runproc/leaf_enforcement_test.go` `allowedImports`, which `TestLeafInvariant_AllowlistOnly` enforces.
   - Covered: cards 2 and 1 cover everything except `allowedImports`. **Partial.**
2. **The `_board` literal moves to the constant in Go code (cards 2–6).**
   - Set: every non-test Go hit (24 files outside fabric plus the 24 fabric files) and every test-file hit, mapped to its enclosing member.
   - Covered: fully. Cards 2/3/5 cover production files, cards 4/6 test members and headers. `livestate_doc_test.go` is exempted by card 7.
3. **No `_board` literal in stencils or skills (card 7).**
   - Set: 18 stencil md files and 2 skill md files; `contracts/specs` has none.
   - Covered: fully by card 7.
4. **Things in Bolt take their place from `BoltDir`/`BoltDirName`, never `BoardDir` (D9 bullet 3; plan cards 2/3).**
   - Set: fabric's own uses (covered by cards 2 and 3), plus production callers outside fabric that build Bolt-checkout paths from `fabricengine.BoardDir`:
     - `internal/tendercli/gate.go:70` and `deps.go:102`: parent-lock store.
     - `internal/orchcli/rules.go:70`, `cmd/lym/stencilseed.go:146`, `internal/tendercli/deps.go:151`: `NewBolt(BoardDir(...))`.
     - `internal/gatecli/report.go:88` and `internal/hubgeom/gateslots.go:17`: `gateslot.Dir`, under `.lym`.
     - `internal/orchroute/paths.go:39`: `_lym/orch/rules.md`.
     - `internal/hubgeom/hubgeom.go:26`: hubreconcile config geometry.
     - `internal/hubgeom/creelgeom.go:19`: creel `.lym`.
     - `ReadShortname(BoardDir(...))` in `hubgeom.go:79,123`, `nearestlive.go:27`, `parent.go:40`, `orchroute/walk.go:40`, `ideengine/spawn.go:159`.
     - `internal/boardengine/config.go:220`: `_lym/config`.
   - Covered: none. Card 5 sweeps only messages and comments; card 6 only tests. **Partial.**
5. **The replay seam is replaced by the registry (card 12).**
   - Set: `cmd/lym/main.go:152`, `boardkit.go:26`, `ownerset_integration_test.go:158` (`TestSetOwners`), `boardengine/replay.go` and `replay_test.go`, `fabricengine/replay.go`, `pull.go`, `pushclassified.go`, `originreconcilemoves.go`, `originreconcile.go`, `bolt.go`, `doc.go`, `replay_integration_test.go`, `boardoverview_test.go`.
   - No md or yaml file outside `_lym` names `InstallBoardReplay`, `ErrBoardReplayMissing` or `FabricReplay`, and no test asserts a `set-aside/board/` tag.
   - Covered: fully by card 12.
6. **`entrymerge` replaces `replayField`/`replayValue` (cards 10 and 11).**
   - Set: `boardengine/replay.go:30-95` and lines 297, 337, 377.
   - Covered: fully by card 11; `TestFabricReplay` is unchanged.
7. **Warp exclude list (card 8): new `warp_exclude` template key and seeding in `wireJunctionSet`.**
   - Set:
     - Config side: `TestConfigTemplate`, `TestLoadConfig`, `TestModules_EveryTemplateKeyClassified`.
     - Wiring callers: add.go:558, reconcile.go:461, checkout.go:133, addanchor.go:154, anchorchange.go:490/507, `fabriccli/clone.go:109`.
     - Config pins in other tests (configsync, hubreconcile, configcli `branch_prefix` rows) assert substrings only, and the sparse base never writes the default.
   - Covered: by card 8; the extra wiring callers inherit the seeding with no edit needed.
8. **New result fields `PullResult.SeamWarning` and `BranchOutcome.Warning`, the `seam_warning` key, and a new `Summary` line (card 9).**
   - Set: `pullResultMap` (no test pins its key set), `pullPairSide` Detail, and the `Summary()` asserts in `originreconcile*_integration_test.go` and `orchcli/start_test.go`. The new line appears only when a warning is set, so existing asserts hold.
   - Covered: by card 9.
9. **`appendPromptText` gains a parameter (card 17).**
   - Set: its one caller at `claudeengine.go:232` (`Prepare`); no test calls it directly. `TestPrepare_AppendSystemPromptMatchesDenyNotice` runs with no reader installed, so the file stays byte-identical.
   - Covered: by card 17.
10. **Root `Long` module list gains `heddle` (card 18).**
    - Set: `cmd/lym/clitree_test.go:151`, `registration_test.go`, and `plugins/ly/skills/cli/SKILL.md`, which the binary generates.
    - Covered: by card 18.
11. **Pre-run installs replace `InstallBoardReplay` (cards 12 and 19).**
    - Set: `cmd/lym/main.go:142-153`.
    - Covered: by cards 12 and 19.

**Findings**

1. **BLOCKING (consistency)** — card 2 `02-bolt-dir-name.md`.
   - Card 2 sets `runproc.HubOwner` from `lymcwd.BoltDirName` and adds the `lymcwd` import. Its targets omit `internal/runproc#allowedImports`, so `TestLeafInvariant_AllowlistOnly` fails at card 2's own gate.
   - Evidence: `internal/runproc/leaf_enforcement_test.go:15-23` allows only `logger`, `proc` and `state`.
   - Missed member: `internal/runproc#allowedImports`, which needs `github.com/Knatte18/loomyard/internal/lymcwd` added.
2. **MEDIUM (design: a method gap)** — cards 3, 5 and 6.
   - D9 says "What lives in Bolt beside the board takes its place from BoltDirName, never from BoardDirName". The sweep moves only fabric's own uses. Production callers outside fabric that build Bolt-checkout paths still go through `fabricengine.BoardDir`.
   - Missed members: the call sites listed in set 4 above. They cover the parent-lock store, Bolt commits, gate-slot `.lym`, `_lym/orch/rules.md`, the hub config geometry, the creel `.lym` and the shortname record.
   - The values are equal today, so nothing breaks now. These sites break once the board stops being Bolt's root.
   - Evidence: `internal/tendercli/gate.go:70`, `internal/orchroute/paths.go:39`, `internal/hubgeom/creelgeom.go:19`, `internal/gatecli/report.go:88`, `cmd/lym/stencilseed.go:146`.

Tool calls:
1. Grep for md/yaml naming the old seam, set-aside tag asserts, `appendPromptText` tests, `Summary`/`pullResultMap` asserts and log-text asserts: the manifest had no reader search for these outputs.
2. Grep for the CLI skill and docs module lists, fabric template-key pins and fabric config docs: this checks the readers of the `Long` list and the new key.
3. Read `plugins/ly/skills/cli/SKILL.md` and searched for pins on the pull-result keys: this checks whether the command index is hand-maintained.
4. Grep for other `FabricReplay.Entries` callers and fabric key pins in configreg, configcli and hubreconcile: these are callers one hop beyond the manifest.
5. Grep for what the `BoardDir` callers outside fabric build: the manifest held per-file counts only, not each call site's purpose.
