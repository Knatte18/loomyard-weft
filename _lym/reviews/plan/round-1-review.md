---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    origin: handler
    location: _lym/plan/18-heddle-cli.md (Create list)
    summary: Card 18 creates only Command() for internal/heddlecli, but PATTERN-cli-cobra requires every module but selfreportcli to expose Command(), RunCLI(out, args) int and RunCLIIn(cwd, out, args) int; every existing cli module but shortcutcli carries RunCLI.
    fix: In 18-heddle-cli.md, add under Create the lines `- plan:internal/heddlecli#RunCLI -> func RunCLI(out io.Writer, args []string) int` and `- plan:internal/heddlecli#RunCLIIn -> func RunCLIIn(cwd string, out io.Writer, args []string) int` (each glyph and head in backticks, as the other Create lines), and add one Intent sentence saying RunCLI runs the subtree from the process cwd and RunCLIIn from the given cwd, as the other cli modules do. The combined overlay with this fix passes validate-plan.
  - id: F2
    severity: BLOCKING
    class: consistency
    origin: lens:affected-set
    location: _lym/plan/02-bolt-dir-name.md (Edit list); internal/runproc/leaf_enforcement_test.go:15-19
    summary: Card 2 gives internal/runproc an internal/lymcwd import for HubOwner, but runproc's allowedImports lists only logger, proc and state, so TestLeafInvariant_AllowlistOnly fails at card 2's own gate.
    fix: In 02-bolt-dir-name.md, add `- internal/runproc#allowedImports` (in backticks) under Edit after `internal/runproc#HubOwner`, and add to the runproc Intent bullet that allowedImports gains `github.com/Knatte18/loomyard/internal/lymcwd`, admitted for the one Bolt directory name, and that leaf_enforcement_test.go's file header names lymcwd in the import set. The combined overlay with this fix passes validate-plan.
  - id: F3
    severity: BLOCKING
    class: design
    origin: lens:repo-invariants
    location: _lym/plan/10-entrymerge-package.md, 13-heddle-store.md, 18-heddle-cli.md (Create lists)
    summary: No card creates a TestMain for the three new test packages internal/entrymerge, internal/heddleengine and internal/heddlecli, so the tier0 guards TestTmuxIsolation_EveryTestPackageRunsThroughTmuxkitMain and the hermetic-env scan (cmd/lym/hermeticenv_test.go) fail at cards 10, 13 and 18.
    fix: Add a Create line `- plan:internal/entrymerge#TestMain -> func TestMain(m *testing.M)` to card 10, `- plan:internal/heddleengine#TestMain -> func TestMain(m *testing.M)` to card 13, and `- plan:internal/heddlecli#TestMain -> func TestMain(m *testing.M)` to card 18 (each in backticks as the other Create lines), and state in each Intent that it lives in an untagged testmain_test.go calling tmuxkit.Main, and for heddleengine and heddlecli, whose tier1 tests spawn git through hubforge, gitkit.HermeticGitEnv() before m.Run, as internal/fabricengine/testmain_test.go does. The combined overlay with these fixes passes validate-plan.
  - id: F4
    severity: BLOCKING
    class: consistency
    origin: lens:repo-invariants
    location: _lym/plan/07-stencil-skill-bolt-dir-sweep.md (TestEnforcement_BoltDirNameDeclaredOnce bullet)
    summary: The new guard walks the whole repository from scankit.Root, yet the card does not give it the //lym:guard marker PATTERN-test-speed requires of a test reading files outside its own package; its sibling scanners in internal/lymcwd/enforcement_test.go carry it.
    fix: In 07-stencil-skill-bolt-dir-sweep.md, in the TestEnforcement_BoltDirNameDeclaredOnce bullet, add that the test carries `//lym:guard` on the line directly above its func line, as TestEnforcement_GeometryLiterals does in the same file.
  - id: F5
    severity: MEDIUM
    class: decision
    origin: handler
    location: _lym/plan/00-overview.md (Shared Decision "the Bolt replay needs its run ledger"); 12-bolt-replayer-registry.md
    summary: The plan departs from D7's step and acceptance criterion that a claimed path with no replayer registered fails naming the registration, replacing it with a failure only when no run ledger is registered; the decision record carries no Added after Discussion entry licensing the departure, and no card carries D7's original wording.
    fix: Ask the parent with lym creel post --parent --expect-reply to record the departure in _lym/discussion/decision-record.md as an Added after Discussion entry (a claimed commit, the ended-set read and the owner read fail with ErrReplayerMissing when no run ledger is registered; a path no registered replayer claims is cherry-picked, since fabric learns a claim only from a registered replayer). Keep the Shared Decision and card 12's rows once the entry exists, citing it in the Shared Decision's Rationale; if the parent refuses, rework card 12 to carry D7's wording as the parent rules.
  - id: F6
    severity: MEDIUM
    class: decision
    origin: handler
    location: _lym/plan/18-heddle-cli.md (Intent, first paragraph)
    summary: The decision record's Constraint says lym heddle is wired through internal/cliwire, but card 18 resolves the hub itself through lymcwd.CwdFrom, lymcwd.Resolve and RequireDrivableWorktree and does not use cliwire; the departure is reasoned in the card but no Added after Discussion entry licenses it.
    fix: Ask the parent with lym creel post --parent --expect-reply to record an Added after Discussion entry stating that lym heddle is hub-only and resolves the hub as the other hub-only CLIs do, since internal/cliwire serves the standalone-capable CLIs and may import neither lymcwd nor fabricengine. Then cite that entry in card 18's first Intent paragraph. If the parent rules otherwise, wire card 18 as the parent rules.
  - id: F7
    severity: MEDIUM
    class: design
    origin: lens:affected-set
    location: _lym/plan/05-module-bolt-dir-sweep.md; internal/tendercli/gate.go:70; internal/orchroute/paths.go:39; internal/hubgeom/creelgeom.go:19
    summary: D9 rules that what lives in Bolt beside the board takes its place from BoltDirName, never from BoardDirName, but the sweep moves only fabric's own uses; production callers outside fabric still build Bolt-checkout paths through fabricengine.BoardDir. Examples are the parent-lock store, NewBolt commits, gate-slot .lym, _lym/orch/rules.md, hub config geometry, the creel .lym and the shortname record.
    fix: "Insert a card after card 6 with lym loom insert-card, slug module-bolt-dir-uses, whose Edit list names each production call site outside internal/fabricengine that builds a Bolt-checkout path from fabricengine.BoardDir: tendercli gate.go and deps.go (parent-lock store, NewBolt), orchcli rules.go, cmd/lym stencilseed.go, gatecli report.go, hubgeom gateslots.go, hubgeom.go, creelgeom.go, nearestlive.go and parent.go, orchroute paths.go and walk.go, ideengine spawn.go, boardengine config.go. Each target is the enclosing member glyph from lym quarry. Its Intent: each call becomes fabricengine.BoltDir; calls that read board.json, overview or design files keep BoardDir; values are unchanged."
  - id: F8
    severity: MEDIUM
    class: consistency
    origin: lens:guard-reach
    location: _lym/plan/09-fabric-worktree-seam.md (Where it fires); internal/fabricengine/parenttips.go:248-283
    summary: Fabric.FastForwardPair fetches origin and ResetHards a checked-out code branch to its upstream, a move D6 says fires took in wherever fabric moves a code branch to origin's commits, yet card 9 neither fires the event there nor states that D6's landing exception covers its Finalize callers.
    fix: "In 09-fabric-worktree-seam.md, add `- internal/fabricengine#Fabric.FastForwardPair` (in backticks) under Edit and a Where-it-fires bullet: after its ResetHard, FastForwardPair fires took in with pathsChangedBetween(warpPath, localHEAD, upstreamSHA), logging the warning only, since FastForwardResult is not printed. Add a TestWorktreeSeam_Events row for it. Otherwise, add a bullet stating that D6's rule that a landing renders nothing covers FastForwardPair, since its only callers are Finalize's parent fast-forwards. The combined overlay with the Edit target passes validate-plan."
  - id: F9
    severity: MEDIUM
    class: consistency
    origin: lens:guard-reach
    location: _lym/plan/08-fabric-exclude-list.md (Tests, Reconcile row); internal/fabricengine/reconcile.go:448-466
    summary: The row expecting Topology.Reconcile to re-seed exclude lines stripped from a pair cannot pass, since repairPairWiring calls wireJunctionSet only when checkJunctionHealth reports the pair unhealthy, and stripping info/exclude lines leaves the junctions healthy.
    fix: In 08-fabric-exclude-list.md, change the Reconcile row to strip the three lines and also break one of the pair's junctions (remove the link) before Topology.Reconcile, then assert the lines are seeded again with the junction repaired; and state in the wireJunctionSet bullet that reconcile seeds the list only for a pair it re-wires, as D5's re-wires wording says.
  - id: F10
    severity: MEDIUM
    class: design
    origin: handler
    location: _lym/plan/18-heddle-cli.md (Intent, Verbs)
    summary: "lym heddle set and remove are new verbs whose text reaches the system prompt of every later Claude session shuttle launches in the hub and the instruction files of every worktree rendered after, yet card 18's Intent never states what they let through or what bounds them. The operator audience annotation is documentation only (internal/clihelp/annotations.go:22-32) and denies nothing."
    fix: In 18-heddle-cli.md, add an Intent paragraph after the verb list stating that set and remove let any lym caller in the hub, an agent included, change the text every later Claude launch appends to its system prompt and every later render writes. State the bound. A write lands only through heddleengine's validation under the Bolt write lock and is committed to Bolt's history by the board sync. A live session or worktree takes it only at its next launch, adoption, compaction after a take-in render, or hand render. No agent deny is added, as the decision record adds none.
  - id: F11
    severity: LOW
    class: design
    origin: lens:guard-reach
    location: _lym/plan/12-bolt-replayer-registry.md (TestBoltReplay_NamesNoBoardFile bullet)
    summary: The guard parses only replay.go and boltreplay.go, while D7's criterion of no board file name or board status in fabric's ended-set and owner-read code also covers weftEndedByOrigin (pull.go), goneEntryEnded (pushclassified.go), addLocalCreatedStamps (originreconcilemoves.go), readOriginOwners (originreconcile.go) and endedByOrigin (bolt.go), so the card's claim that no board file name returns there is unguarded.
    fix: In 12-bolt-replayer-registry.md, widen TestBoltReplay_NamesNoBoardFile to parse every function declaration the card's Edit list names in pull.go, pushclassified.go, originreconcilemoves.go, originreconcile.go and bolt.go besides replay.go and boltreplay.go, and fail on the same literals inside those functions' bodies. Otherwise, narrow the card's sentence to replay.go and boltreplay.go only.
  - id: F12
    severity: LOW
    class: consistency
    origin: lens:guard-reach
    location: _lym/plan/08-fabric-exclude-list.md (wireJunctionSet bullet and the unparseable-config row); internal/fabricengine/junction.go:115-139
    summary: The card seeds the list after the junction lines without saying the config loads before seedLymJunction, while its row asserts an unparseable config fails WireJunctions and seeds nothing; loaded at the seeding point, the links and junction exclude lines are already written.
    fix: In 08-fabric-exclude-list.md, state in the wireJunctionSet bullet that LoadConfig(ConfigDirs(l)) runs first in wireJunctionSet, before seedLymJunction, so a load failure writes nothing; the seeding itself stays after the junction lines.
  - id: F13
    severity: LOW
    class: consistency
    origin: lens:guard-reach
    location: _lym/plan/18-heddle-cli.md (pre-run paragraph and TestHeddleCLI rows)
    summary: The card prescribes that a cwd RequireDrivableWorktree refuses (the Bolt checkout, a records sibling) keeps that function's text, but no TestHeddleCLI row runs a verb from the Bolt checkout or a records sibling.
    fix: In 18-heddle-cli.md, add a TestHeddleCLI row running list from the hub's Bolt checkout and from the prime's records sibling, each refusing with RequireDrivableWorktree's text and writing nothing.
  - id: F14
    severity: LOW
    class: consistency
    origin: lens:repo-invariants
    location: _lym/plan/12-bolt-replayer-registry.md (Uses list); internal/fabricengine/destroy.go:1985
    summary: restoreReplayTip now restores after any replayer's failure, yet its message still reads after a failed board replay and card 12 lists it only under Uses.
    fix: In 12-bolt-replayer-registry.md, move `internal/fabricengine#restoreReplayTip` (in backticks) from Uses to Edit and add to the replay.go bullets that its message says after a failed Bolt replay. The combined overlay with this move passes validate-plan.
  - id: F15
    severity: LOW
    class: consistency
    origin: lens:repo-invariants
    location: _lym/plan/01-pattern-entries.md (Edit list); pattern/PATTERN-config-layers.md:31; pattern/PATTERN-cwd-resolution.md:42; pattern/PATTERN-fabric-git.md:25; pattern/PATTERN-durable-vs-ephemeral-state.md:7
    summary: Background files still place Bolt-checkout things through BoardDir or the _board literal (the machine config file, the hub-only containers, the weft-side .gitignore writer, the .lym sibling in a hub), which D9 moves to BoltDir and the Bolt checkout, and no card lists them.
    fix: In 01-pattern-entries.md, add `//pattern/PATTERN-config-layers.md`, `//pattern/PATTERN-cwd-resolution.md` and `//pattern/PATTERN-fabric-git.md` (in backticks) under Edit. Add an Intent paragraph making each of those lines, and PATTERN-durable-vs-ephemeral-state.md's line 7, say BoltDir or the Bolt checkout instead of BoardDir or _board. The combined overlay with these targets passes validate-plan.
  - id: F16
    severity: LOW
    class: consistency
    origin: lens:repo-invariants
    location: _lym/plan/08-fabric-exclude-list.md; internal/fabricengine/config.go:1-8
    summary: config.go's file header still says Config carries only the branch prefix and the pathspec, which card 8 falsifies by adding warp_exclude, and no card lists the header.
    fix: Make card 8's Intent say that config.go's file header names the third key beside the ConfigSpec comment. Adding the file `internal/fabricengine/config.go` to card 8's targets is refused by redundant-file-target, since the card already lists internal/fabricengine#Config and internal/fabricengine#ConfigSpec. So the header edit rides on the card's existing config.go members, or the fixer defers it with that check as the reason.
  - id: F17
    severity: LOW
    class: decision
    origin: handler
    location: _lym/plan/04-fabric-test-bolt-dir-sweep.md (livestate_doc_test.go bullet); 07-stencil-skill-bolt-dir-sweep.md (exemptions); internal/fabricengine/livestate_doc_test.go:189
    summary: The plan leaves the _board literal in a livestate_doc_test.go comment and exempts it from the new guard, while D9 rules a comment names the Bolt checkout and its acceptance criterion allows the literal only in the one declaration and TestEnforcement_GeometryLiterals.
    fix: In 04-fabric-test-bolt-dir-sweep.md, replace the livestate_doc_test.go bullet with one making the quoted error text at line 189 read with the Bolt directory name elided (for example no <Bolt directory> entry and no records sibling). List that file's members under Edit as the card does for other header comments. In 07-stencil-skill-bolt-dir-sweep.md, drop the livestate_doc_test.go exemption.
  - id: F18
    severity: LOW
    class: design
    origin: handler
    location: _lym/plan/14-heddle-render.md (Create list, ExcludeWayForward)
    summary: fabricengine.ExcludeWayForward is part of fabric's public surface and reused across packages, yet card 14 creates it inside heddle's render card instead of on the fabric card that owns the warp_exclude key.
    fix: Move `- plan:internal/fabricengine#ExcludeWayForward -> func ExcludeWayForward(path string) string` (in backticks) from card 14's Create list to card 8's Create list, and move its Intent sentence and its fabricengine/doc.go note to card 8. List `plan:internal/fabricengine#ExcludeWayForward` under card 14's Uses. The combined overlay with this move passes validate-plan.
  - id: F19
    severity: LOW
    class: design
    origin: handler
    location: _lym/plan/10-entrymerge-package.md (TestTable rows)
    summary: Four TestTable rows duplicate behaviour TestFabricReplay (tier0) already covers through the board over the same merge. An applied field duplicates DifferentEntriesChangedOnTheTwoSidesBothLand, a both-sides field lost SameFieldChangedOnBothSidesKeepsOriginsValue, a field origin already holds CommitOriginAlreadyHoldsIsEmpty, and Same false on one differing field RemovalOfAnEntryOriginChangedIsLost.
    fix: In 10-entrymerge-package.md, cut those four rows from TestTable and keep only the rows no existing test covers. Those are the origin value equal to the parent's counting as unchanged, Changed treating nil and empty alike through the getter, and AddClash reporting each differing field. Name TestFabricReplay as the cover for the rest.
  - id: F20
    severity: LOW
    class: design
    origin: handler
    location: _lym/plan/06-module-test-bolt-dir-sweep.md (Intent, first bullet)
    summary: The card leaves to its run which packages cannot import fabricengine without a cycle and so take lymcwd.BoltDirName, a fact known now. fabricengine depends on internal/yamlengine and internal/vscode, whose in-package tests TestSetValues and TestBuildWorkspace the card lists.
    fix: In 06-module-test-bolt-dir-sweep.md, replace the sentence about a package that cannot import fabricengine with one naming internal/yamlengine#TestSetValues and internal/vscode#TestBuildWorkspace as the members that take lymcwd.BoltDirName, being in-package tests of packages fabricengine imports, and every other listed member as taking fabricengine.BoltDirName.
---

### [BLOCKING] heddlecli lacks RunCLI and RunCLIIn

**Location:** `_lym/plan/18-heddle-cli.md` (Create list)

**Issue:** PATTERN-cli-cobra's background requires each module to expose `Command()`, `RunCLI(out io.Writer, args []string) int` and, for every module but `internal/selfreportcli`, `RunCLIIn(cwd, out, args) int`. Every existing `internal/*cli` package but `shortcutcli` declares `RunCLI`. Card 18 creates only `Command()`, `commandOf` and the verb builders. That breaks a listed PATTERN entry, so the finding is BLOCKING.

### [BLOCKING] runproc's import allowlist misses lymcwd

**Location:** `_lym/plan/02-bolt-dir-name.md`; `internal/runproc/leaf_enforcement_test.go:15-19`

**Issue:** Card 2 sets `runproc.HubOwner` from `lymcwd.BoltDirName`. `TestLeafInvariant_AllowlistOnly` calls `scankit.AssertImportAllowlist` over `allowedImports`, which holds only `logger`, `proc` and `state`. The card cannot pass its own gate, and no card lists `allowedImports`. The repo-invariants lens reported the same defect.

### [BLOCKING] The new test packages have no TestMain

**Location:** cards 10, 13, 18

**Issue:** PATTERN-test-isolation says every test package runs through `tmuxkit.Main`, and that a package spawning git calls `gitkit.HermeticGitEnv()` in its `TestMain`. `cmd/lym/tmuxisolation_test.go` (tier0, `//lym:guard`) and `cmd/lym/hermeticenv_test.go` (tier0) enforce both. `internal/entrymerge`, `internal/heddleengine` and `internal/heddlecli` gain test files with no `TestMain` planned.

### [BLOCKING] The repository-wide guard carries no //lym:guard

**Location:** `_lym/plan/07-stencil-skill-bolt-dir-sweep.md`

**Issue:** `TestEnforcement_BoltDirNameDeclaredOnce` reads every `.go` file and the md files under `contracts/` and `plugins/` from `scankit.Root(t)`. PATTERN-test-speed requires `//lym:guard` directly above such a test's func line, so a narrowed gate runs it. The card names the file and the tag but not the marker.

### [MEDIUM] The run-ledger Shared Decision departs from D7 without a license

**Location:** `_lym/plan/00-overview.md` (Shared Decision "the Bolt replay needs its run ledger"); card 12

**Issue:**
- D7 says a commit whose claimed paths have no replayer registered fails naming the registration, and its acceptance criterion repeats it.
- The Shared Decision says outright that the plan departs from that wording. It fails only a claimed commit, an ended-set read or an owner read with no run ledger registered, and it cherry-picks any path no registered replayer claims.
- The reasoning is sound: fabric cannot know a path is claimed without a registered replayer, and naming `heddle.json` in fabric breaks D7 and PATTERN-heddle-instructions.
- The decision record has no `Added after Discussion` entry, so the departure is unlicensed.
- No card carries D7's original acceptance; card 12's `TestBoltReplayers` rows assert the plan's version.

### [MEDIUM] Card 18 departs from the cliwire Constraint

**Location:** `_lym/plan/18-heddle-cli.md`

**Issue:** The record's Constraints say "`lym heddle` is one cobra subtree, wired through `internal/cliwire`." Card 18 deliberately wires through `lymcwd` and `RequireDrivableWorktree` instead. `internal/cliwire/doc.go` confirms that cliwire serves only the standalone-capable CLIs and may import neither `lymcwd` nor `fabricengine`, so the card's reading is defensible. It still contradicts a Constraint with no recorded license.

### [MEDIUM] Bolt-checkout paths outside fabric still come from BoardDir

**Location:** cards 3, 5, 6; e.g. `internal/tendercli/gate.go:70`, `internal/orchroute/paths.go:39`, `internal/hubgeom/creelgeom.go:19`, `internal/gatecli/report.go:88`, `cmd/lym/stencilseed.go:146`

**Issue:** D9's third bullet is general: what lives in Bolt beside the board takes its place from `BoltDirName`, never from `BoardDirName`. Card 3 moves fabric's own uses. Cards 5 and 6 change only messages, comments and tests. Some production callers build `.lym`, `_lym` config, rules, lock and shortname paths from `fabricengine.BoardDir`, and they stay on the board name. One `design` finding covers the method gap.

### [MEDIUM] FastForwardPair takes in origin's commits without firing took in

**Location:** `_lym/plan/09-fabric-worktree-seam.md`; `internal/fabricengine/parenttips.go:248-283`

**Issue:** `FastForwardPair` runs `f.code.Fetch()`, then `f.ResetHard(rec, upstreamSHA)` on the code worktree. Its callers are `landingshed`'s `finalize.go:379` and `finalize_pr.go:247`. D6 fires "took in" wherever fabric moves a checked-out code branch to origin's commits, and also says a landing renders nothing. Card 9 says nothing either way, so the implementer cannot tell which rule governs this site.

### [MEDIUM] The Reconcile re-seed row cannot pass

**Location:** `_lym/plan/08-fabric-exclude-list.md`; `internal/fabricengine/reconcile.go:448-466`

**Issue:** `repairPairWiring` reaches `wireJunctionSet` only under `if !junctionHealthy`. A pair whose exclude lines were stripped still has healthy junctions, so `Topology.Reconcile` reports it `already_healthy` and seeds nothing. As the card writes the row, it fails.

### [MEDIUM] lym heddle set states no attack-surface bound

**Location:** `_lym/plan/18-heddle-cli.md`

**Issue:** `set` and `remove` change text that card 17 appends to every later Claude session's system prompt and that cards 14 and 15 render into every worktree's instruction files. The `operator` audience annotation is a help-text tag (`internal/clihelp/annotations.go`) and no deny, so any agent in the hub can run the verb. The Intent states neither what this lets through nor what bounds it.

### [LOW] The no-board-file guard scans two files only

**Location:** `_lym/plan/12-bolt-replayer-registry.md`

**Issue:** The ended-set and owner readers card 12 rewrites live in `pull.go`, `pushclassified.go`, `originreconcilemoves.go`, `originreconcile.go` and `bolt.go`. `TestBoltReplay_NamesNoBoardFile` parses only `replay.go` and `boltreplay.go`, yet the card says it stops a board file name from returning to "the replay, ended-set and owner-read code".

### [LOW] Card 8 leaves the config load order open

**Location:** `_lym/plan/08-fabric-exclude-list.md`; `internal/fabricengine/junction.go:115-139`

**Issue:** `wireJunctionSet` runs `seedLymJunction`, `seedWeftArtifactExcludes`, then `seedGitExclude`. The card puts the new seeding after the junction lines but does not say where the config loads. The unparseable-config row's "seeds nothing" holds only if the load comes first.

### [LOW] RequireDrivableWorktree's refusal has no reaching row

**Location:** `_lym/plan/18-heddle-cli.md`

**Issue:** The pre-run paragraph prescribes the refusal for the Bolt checkout and a records sibling, but `TestHeddleCLI` names no row run from either.

### [LOW] restoreReplayTip keeps a board-replay message

**Location:** `_lym/plan/12-bolt-replayer-registry.md`; `internal/fabricengine/destroy.go:1985`

**Issue:** After card 12, the restore follows any replayer's failure, yet the message still says "after a failed board replay". The card lists the function under Uses, so it cannot change it.

### [LOW] Pattern background files still place Bolt things on the board name

**Location:** `_lym/plan/01-pattern-entries.md`; `pattern/PATTERN-config-layers.md:31`, `pattern/PATTERN-cwd-resolution.md:42`, `pattern/PATTERN-fabric-git.md:25`, `pattern/PATTERN-durable-vs-ephemeral-state.md:7`

**Issue:** These lines place the machine config file, the hub-only containers, the weft-side `.gitignore` writer and the hub's `.lym` sibling under `BoardDir` or `_board`. Under D9 those are the Bolt checkout. No card updates them, and card 7's guard does not scan `pattern/`.

### [LOW] config.go's header goes stale

**Location:** `_lym/plan/08-fabric-exclude-list.md`; `internal/fabricengine/config.go:1-8`

**Issue:** The header says the file defines the branch prefix and the pathspec. Card 8 adds a third key and updates only the `ConfigSpec` comment. `redundant-file-target` refuses listing the file beside the card's `config.go` members, so the edit has to ride on those members.

### [LOW] A _board literal survives in livestate_doc_test.go

**Location:** cards 4 and 7; `internal/fabricengine/livestate_doc_test.go:189`

**Issue:** The hit is a comment quoting an earlier error text. D9 says a comment names "the Bolt checkout", and its acceptance criterion allows the literal only in the one declaration and `TestEnforcement_GeometryLiterals`. Exempting the file from the guard keeps a departure the record does not license.

### [LOW] Card 14 creates fabric's ExcludeWayForward

**Location:** `_lym/plan/14-heddle-render.md`

**Issue:** `ExcludeWayForward` is an exported fabric function, and the rubric gives public surface reused across packages its own card. It belongs with card 8, the card that owns `warp_exclude`. Heddle should consume it, not declare it.

### [LOW] TestTable repeats TestFabricReplay's rows

**Location:** `_lym/plan/10-entrymerge-package.md`

**Issue:** Card 11 runs the board replay over `entrymerge` with `TestFabricReplay` unchanged. Four of `TestTable`'s rows then pin behaviour that test already pins: `DifferentEntriesChangedOnTheTwoSidesBothLand`, `SameFieldChangedOnBothSidesKeepsOriginsValue`, `CommitOriginAlreadyHoldsIsEmpty` and `RemovalOfAnEntryOriginChangedIsLost`.

### [LOW] Card 6 leaves the import-cycle packages to be found at run time

**Location:** `_lym/plan/06-module-test-bolt-dir-sweep.md`

**Issue:** `go list -deps ./internal/fabricengine` includes `internal/yamlengine` and `internal/vscode`. The card lists their in-package tests `TestSetValues` and `TestBuildWorkspace`, but it only says some package "that cannot import `fabricengine` without a cycle" takes `lymcwd.BoltDirName`.

## Rejected

- Card 8's `warp_exclude` attack-surface statement: the card states its bound, Cross-pair with the local layer refused, and what it does, exclusion through `info/exclude` and never a junction. That meets the rubric.
- The clone's "made" firing mid-`CloneAndWire` rather than last: the steps after `WireJunctionsWith` touch weft and Bolt only. D6's conditions (wired, lines seeded, Bolt present) hold at the firing point, so nothing a render reads changes after it.
- The guard-reach lens's remark that `lymcwd.Resolve` may refuse first with `ErrCwdOutsideAnchor` on a subpath hub: the error it cites came from a subdirectory of a checkout, not from a worktree root, so it does not show the refusal is unreachable. The kept part of that report is F13.
- The fabric tests naming `heddle.json`, card 16's `TestRecordsReplay` rows: D7's acceptance requires replay rows over `heddle.json`, and `ReplayBoltForTest` is visible only to fabric's own tests. PATTERN-heddle-instructions' "fabric names no heddle file" binds fabric's code, which stays clean.

## Focus directive verdicts

- Run-ledger Shared Decision: unlicensed departure, and no card carries D7's original acceptance (F5).
- D1–D9 and Constraints: carried except D9's third bullet outside fabric (F7), D9's comment rule in one test file (F17), D7's acceptance (F5) and the cliwire Constraint (F6). The sweep cards and card 12 add no unlicensed scope.
- Card order 8–19: no card uses a symbol a later card creates.
- Refusal reach: F9, F12 and F13. Every other refusal on cards 8, 9, 12, 13, 17 and 18 has a reaching input and row.
- Attack surface: F10. Cards 9, 12, 15, 17 and 19 state their bounds.
- Test economy: F19. Cards 4 and 6 add no test, and card 7's guard pins a rule across a stencil family, not static prose.
- Granularity: F18. Cards 2–7 and 13–16 split no private helper from its type.
- Edit-card glyphs and re-sign arrows: held. Card 3's and card 5's file targets cover header comments outside any member, and every changed signature carries the arrow.
- PATTERN fit: F1, F2, F3 and F4. Fabric-vocabulary naming holds across cards 8, 9 and 13–19.
- Research steps, board edits, filing tasks: F20. No card edits the task's entry or files a task.
- Verify lines: no card carries one.
