---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/plan/10-status-seams-spawn-detached-push.md (PATTERN.md edit), _lyx/plan/15-command-engine-replaces-switched-rows.md (PATTERN.md edits)
    summary: The declared PATTERN.md growth crosses the overview's headroom line at card 10's card gate, and no card cuts bytes to make room, so the headroom lint refuses the plan.
    fix: Add a step to card 10's Intent, ahead of its PATTERN-push-both-sides edit, that cuts PATTERN.md by at least the whole plan's growth before cards 10 and 15 add their text. The growth is the push-both-sides lengthening (about 69 bytes), the recipe-setting-lookup lengthening and the new PATTERN-command-row line, about 370 bytes in total, and the result must stay at or under internal/pattern's HeadroomBytes (15564) against the merge base. Cut only wording, outside the three declared entries, that changes no entry's rule, and confirm the result with the card gate's headroom lint. If no such cut exists, card 10 asks ly:orch how to make room before editing PATTERN.md, since the decision record declares only these three PATTERN changes.
    origin: handler
  - id: F2
    severity: BLOCKING
    class: consistency
    location: _lyx/plan/15-command-engine-replaces-switched-rows.md (plan:internal/shedrecipe#commandEntry, plan:internal/shedrecipe#TestCommandEntry)
    summary: commandEntry's build-time refusals are specified only as naming the row and the key, with no way-forward clause, and TestCommandEntry does not assert their messages; PATTERN-refusal-way-forward binds internal/shedrecipe.
    fix: "In card 15's Registry paragraph, say that each build-time refusal (an unknown config key, a missing or malformed switch or command reference, a non-int or non-positive timeout_s) ends with a `way forward:` clause naming the fix in the recipe row. Examples: `way forward: set switch and command to module.key setting references in the row's config`, and `way forward: set timeout_s to a positive integer of seconds or remove it`. Change the TestCommandEntry bullet so that each refusal row asserts the message carries the row, the key and its `way forward:` clause."
    origin: lens:way-forward
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/plan/07-status-push-spawn-entry.md (plan:internal/fabricengine#statusPushRequest, internal/tendercli#InstallNetworkGate, cmd/lyx#newRoot), _lyx/plan/09-pair-teardown-lock.md (plan:internal/fabricengine#DetachedPush)
    summary: On a hub with a subpath anchor, lyxcwd.Resolve from the child's `_board` working directory fails the cwd anchor gate. The child then installs no pause gate, arms no log sink and cannot resolve the hub DetachedPush is handed, which contradicts card 7's claim that the pre-run still resolves the hub.
    fix: "Card 7 no longer derives the child's hub from its working directory. Two options: (a) InstallNetworkGate, when given a non-nil pause slug, resolves its location through lyxcwd.ResolveWorktree over the working directory, which is the `_board` worktree root, instead of the gated Resolve; or (b) the child is told the hub explicitly, derived in fabric from the told records path (filepath.Dir of WeftPath). Card 9's fabriccli bypass hands DetachedPush that same told hub, not one resolved from the cwd. Card 7 also says how the child's log sink reaches fabricengine.HubLogsDir on such a hub: logger.SetDurableSinkDir(fabricengine.HubLogsDir(hub)) in the bypass. TestSpawnStatusPush, or the tendercli gate test, gains a row over a hubforge hub of Shape{Anchor: \"backend\"} asserting the gate installs under the told slug."
    origin: lens:guard-reach
  - id: F4
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/07-status-push-spawn-entry.md (plan:internal/fabricengine#pushSpawnCommand, plan:cmd/lyx#pauseSlugOverride)
    summary: A prime push with an empty gate slug carries no --pause-slug, so pauseSlugOverride returns nil and the gate falls back to PauseGeometry over `_board`. `_board` is a linked worktree, so the child waits under a pause entry named `_board` rather than the prime's, contrary to D1's empty slug.
    fix: "Card 7's request carries the pause slug whenever SpawnStatusPush sets it, the empty prime slug included. Either pushSpawnCommand always writes --pause-slug (as `--pause-slug=` when empty) and pauseSlugOverride returns a pointer whenever the flag was set, empty value included, or InstallNetworkGate treats a child whose cwd is the `_board` worktree as the prime. TestPushSpawnCommand's empty-PauseSlug row and TestPauseSlugOverride gain the set-to-empty case, and the tendercli gate test's new step adds a row installed from `_board` with an empty pause slug, landing under slug \"\"."
    origin: lens:guard-reach
  - id: F5
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/09-pair-teardown-lock.md (internal/fabricengine#Topology.Remove)
    summary: The respawn after a failed Remove calls SpawnStatusPush with Remove's own location, which is the prime (pairteardown passes prime). That requests the prime's records-only push with no slugs, not the pair's push the card's test row expects. The card also narrows D1's leaves-the-pair to leaves-the-task-worktree.
    fix: "Card 9 says the respawn builds the pair's location from the hub and slug, through lyxcwd.ResolveWorktree(WorktreePath(l, slug)), and calls SpawnStatusPush(pairLocation, \"\"), so the request carries both of the pair's worktrees and both slugs. When the task worktree is already gone but the records worktree remains, as when removeSibling fails after the task teardown, the respawn requests the records side alone, carrying both slugs. The condition becomes D1's: a Remove that fails after taking the lock and leaves either of the pair's worktrees. TestRemove_TeardownLockAgainstThePairsPush's failed-archive row asserts the recorded request names the pair's worktree paths."
    origin: lens:guard-reach
  - id: F6
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/07-status-push-spawn-entry.md (internal/fabricengine#TestSpawnPairPush_HandsTheDetachedPushBothSidesOfThePair)
    summary: After card 7, SpawnPairPush goes through SpawnStatusPush, which resolves the prime and reads branch marks first. The test builds a bare Location under t.TempDir with no repository, so swapping the seam cannot keep it passing, and its behaviour is TestSpawnStatusPush's task-pair row.
    fix: "Move internal/fabricengine#TestSpawnPairPush_HandsTheDetachedPushBothSidesOfThePair from card 7's Edit list into a new Delete group on card 7, and drop it from the Tests bullet that swaps the re-typed seam. Card 7's TestSpawnStatusPush task-pair row also calls SpawnPairPush on the same pair and asserts it records the same request and returns the seam's error. This shape passes the plan gate."
    origin: lens:affected-set
  - id: F7
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/05-fabric-remove-local-pair.md (plan:internal/fabricengine#TestOriginWrites_LocalMarkedBranch)
    summary: The test is planned as an internal (package fabricengine) test over a hubforge hub, but an internal fabricengine test cannot import hubforge without an import cycle (internal/fabricengine/export_test.go:148-152).
    fix: "In card 5, change the handle to plan:internal/fabricengine_test#TestOriginWrites_LocalMarkedBranch, add internal/fabricengine#ArchiveWeftTipForTest and internal/fabricengine#CheckRemoteBranchRequestForTest to Uses, and reword its Tests paragraph. The test is an external test that drives archiveWeftTip through ArchiveWeftTipForTest and the gate through CheckRemoteBranchRequestForTest over the AddLocal pair. This shape passes the plan gate."
    origin: handler
  - id: F8
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/04-fabric-add-local.md (plan:internal/fabricengine#ErrLocalAddOverOrdinary, plan:internal/fabriccli_test#TestRunCLI_LocalPairScenario)
    summary: The refusal's way forward, plain lyx fabric add over the branch, cannot succeed for the card's own reaching input, an unmarked warp branch, which plain Add refuses with ErrBranchExists (add.go:104). The CLI step over an ordinary pair's slug hits the existing-directory refusal first, whose message does not name lyx fabric add.
    fix: "In card 4, make the way forward depend on the branch that tripped the refusal. For an unmarked warp branch, name ErrBranchExists's remedies: check the pair out with lyx fabric checkout <slug> if it is live, delete the leftover branch, or use another slug. For an unmarked weft branch or an origin remote-tracking ref, keep `lyx fabric add <slug>`. Each TestAddLocal refusal row asserts its own way forward. TestRunCLI_LocalPairScenario's ordinary-slug step uses a slug whose ordinary pair's worktrees were removed by hand, leaving its warp branch, so the local refusal fires rather than the existing-directory refusal."
    origin: lens:way-forward
  - id: F9
    severity: MEDIUM
    class: design
    location: _lyx/plan/08-push-lock-pending-flags.md (plan:internal/fabricengine#markPending, plan:internal/fabricengine#takePending, plan:internal/fabricengine#coalesceFlagged)
    summary: Card 8 adds a try-lock and pending flags whose content is a whole PushSpawn a holder respawns as written. Its Intent states no bound on what a flag file can make a holder spawn (any path, any pause or teardown slug), nor what the try-lock now lets through, a hung holder leaving every child's commits waiting.
    fix: "Add a what-this-lets-through / what-bounds-it statement to card 8's Intent. What it lets through: a child that finds the lock held exits and leaves its commits to the holder, so a holder that hangs delays every later push of that records worktree until it exits; and a holder respawns whatever request a flag holds. What bounds it: flags live only in the records worktree's untracked `.weft/` lock directory, and only markPending writes them; takePending accepts a flag only when its side's path is the worktree that lock directory belongs to (the records worktree for records, its paired code worktree for code), refusing any other with a Warn; and the hung-holder residual is the one D1 already accepts. TestCoalesceFlagged_PendingFlags gains a row where a flag naming a foreign path is dropped with a Warn and spawns nothing."
    origin: handler
  - id: F10
    severity: MEDIUM
    class: design
    location: _lyx/plan/00-overview.md (## verify:)
    summary: Cards 10 and 15 edit PATTERN.md, and cards 1, 4, 10, 15 and 16 edit docs/, but verify runs neither internal/pattern (whose overview test checks PATTERN.md's real size and shape) nor internal/lyxcwd (whose docslink test checks every docs/ link).
    fix: "Append ./internal/pattern ./internal/lyxcwd to both go test lines of the overview's verify block, the untagged one and the -tags integration one. This shape passes the plan gate."
    origin: handler
  - id: F11
    severity: LOW
    class: consistency
    location: _lyx/plan/12-ide-seed-and-open-verbs.md (internal/idecli#Command)
    summary: Card 12 adds two new verbs, but its Intent states neither what they let through nor what bounds them (rubric attack-surface question).
    fix: "Add a sentence to card 12's Intent. What it lets through: an operator or a Command row can write a pair's .vscode config and title-bar colour, or start VS Code, on any existing pair of the hub, by slug. What bounds it: the slug resolves only through fabricengine.WorktreePath to a worktree of this hub; seed writes only .vscode/ and its exclude line; open starts only ideengine.CodeLauncher on that folder."
    origin: handler
  - id: F12
    severity: LOW
    class: consistency
    location: _lyx/plan/14-command-producer.md (plan:internal/shedadapters#CommandProducer.Call, step 3)
    summary: A command key that cannot be read or is not a string gets step 2's correct-it-then-rerun error. The command is read only from the template and the board default layer, so a correction in the task worktree's batten.yaml is ignored and the re-run fails again.
    fix: "In card 14's step 3, the command-key error names where to correct the key: `lyx config set --layer default <module>.<key>` from the prime, then <Rerun>. TestCommandProducer's command-lookup-error and non-string rows assert that text."
    origin: lens:way-forward
  - id: F13
    severity: LOW
    class: consistency
    location: _lyx/plan/08-push-lock-pending-flags.md (internal/fabricengine#acquirePushLock, internal/fabricengine#PushPairAnchored)
    summary: A synchronous pusher that misses its bounded wait on coalesceFlagged's re-acquire, after its own push already landed, returns today's ErrPushLockBusy, whose text says neither side was pushed. commit-records then reports a failed push that in fact landed.
    fix: "In card 8, a synchronous pusher returns ErrPushLockBusy only when its first acquire misses. A missed re-acquire after a completed step leaves the flag to the current holder and returns nil. TestPushLock_BoundedWaitGivesUpAndUnboundedWaitBlocks gains a row pinning that."
    origin: lens:way-forward
  - id: F14
    severity: LOW
    class: consistency
    location: _lyx/plan/05-fabric-remove-local-pair.md (internal/fabricengine#checkRemoteBranchRequest)
    summary: "The new local-only CheckOwnership refusal names Add's leftover replacement as its production input. A leftover is a branch present only on origin, so no local mark exists there, and the refusal names no next step."
    fix: "In card 5, drop the Add leftover-replacement claim and state the check as a second line behind Remove's local path and Cleanup's mark read (card 6), reached by TestOriginWrites_LocalMarkedBranch. End the refusal's reason with its next step: the pair is local-only, so remove it with lyx fabric remove <slug>, which runs no origin step."
    origin: lens:guard-reach
  - id: F15
    severity: LOW
    class: scope
    location: _lyx/plan/10-status-seams-spawn-detached-push.md (Edit list)
    summary: "Card 10 changes loomcli's commitStatusDeps.Commit to return a committed flag, but internal/loomcli/drivewiring_test.go's TestLoomPark builds commitStatusDeps with Commit: func(string) error and is not listed."
    fix: "Add internal/loomcli#TestLoomPark to card 10's Edit list, and add to its Tests paragraph that TestLoomPark takes the new Commit shape with its park assertions unchanged. This shape passes the plan gate."
    origin: lens:affected-set
  - id: F16
    severity: LOW
    class: consistency
    location: _lyx/plan/15-command-engine-replaces-switched-rows.md (plan:internal/battencli#runRowCommand)
    summary: runRowCommand goes into land.go only so the existing verifyRunAllowed entry admits it, while that entry's reason still says land.go runs the landing config's deploy command alone (cmd/lyx/verifiedtree_test.go:26).
    fix: "Add cmd/lyx#verifyRunAllowed to card 15's Edit list, and add to its Intent that the internal/battencli/land.go reason now names both the landing deploy command and the Command rows' commands, which verify no tree. This shape passes the plan gate."
    origin: lens:affected-set
  - id: F17
    severity: NIT
    class: consistency
    location: _lyx/plan/15-command-engine-replaces-switched-rows.md (internal/battencli/cli.go:396)
    summary: "The --window envelope's terminal_window text says the window opens through the resolved launcher at the Open-Window row. Once the row runs open_window_command, that holds only for the default command."
    fix: "In card 15, add a sentence that the --window envelope's terminal_window text says the Open-Window row runs open_window_command, whose default opens through the named launcher, in the same edit as the --window and --open-terminal flag help."
    origin: lens:affected-set
---

### [BLOCKING] PATTERN.md growth crosses the headroom line

**Location:** cards 10 and 15, their `PATTERN.md` edits.
**Issue:** `PATTERN.md` is 15556 bytes at the merge base, and `internal/pattern/check.go:24` sets `HeadroomBytes` to 95% of the 16384-byte cap, 15564 bytes.
`internal/pattern/headroom.go:11` refuses any growth that ends past that line, and the card gate runs this headroom lint (`pattern/PATTERN-test-speed.md`, "Gate per tier").
Card 10's push-both-sides lengthening alone adds about 69 bytes, and card 15 then adds more.
No card cuts anything, so card 10's gate refuses.

### [BLOCKING] commandEntry's build-time refusals carry no way forward

**Location:** card 15, `plan:internal/shedrecipe#commandEntry` and `plan:internal/shedrecipe#TestCommandEntry`.
**Issue:** Card 15 specifies the refusals only as "naming the row and the key", and `TestCommandEntry` asserts only that the key is named.
`pattern/PATTERN-refusal-way-forward.md` binds `internal/shed*`: each refusal needs a trailing `way forward:` clause and a test that asserts its message.
The existing entries carry one (`internal/shedrecipe/entries_settings.go:46`, `:49`).

### [MEDIUM] Children started in `_board` break on subpath-anchored hubs

**Location:** card 7, root pre-run and `InstallNetworkGate`; card 9, the bypass's `DetachedPush`.
**Issue:** `checkCwdAnchorGate` (`internal/lyxcwd/anchor.go:134-140`) requires the cwd to equal `<worktree>/<anchor>`.
From `<hub>/_board` on a hub with a recorded subpath anchor, `lyxcwd.Resolve` fails.
`InstallNetworkGate` then returns before installing anything (`internal/tendercli/gate.go:31-33`), and the sink does not arm (`internal/logger/sink.go:182`).
Card 9's bypass resolves the hub from the same cwd.
hubforge supports such hubs (`Shape{Anchor: "backend"}`), and no card covers them.

### [MEDIUM] An empty pause slug lands under a `_board` pause entry

**Location:** card 7, `pushSpawnCommand` and `pauseSlugOverride`.
**Issue:** Card 7 omits an empty `--pause-slug`, and `pauseSlugOverride` returns nil for an unset flag.
The gate then uses `PauseGeometry` over `_board`.
`_board/.git` is a gitdir file into `.../worktrees/_board`, so `isPrimeWorktree` reads false (`internal/hubgeom/hubgeom.go:74`), and `pauseGeometry` takes the slug `_board` (`internal/hubgeom/pausegeom.go:28-30`).
This reaches batten cleanup's push, prime `Fabric.Commit` and `lyx fabric sync`.

### [MEDIUM] Remove's respawn pushes the prime, not the pair

**Location:** card 9, `Topology.Remove`.
**Issue:** `pairteardown` calls `top.Remove(prime, ...)` (`internal/pairteardown/teardown.go:138`).
Card 7's `SpawnStatusPush` builds the prime's records-only request with no slugs from that location.
The card's own failed-archive row expects "one push request for the pair with both slugs".
The card also respawns only while the task worktree stands.
D1 asks for a respawn whenever the pair is left, which includes `removeSibling` failing with the records worktree still present.

### [MEDIUM] TestSpawnPairPush cannot survive card 7 and duplicates its replacement

**Location:** card 7, `internal/fabricengine#TestSpawnPairPush_HandsTheDetachedPushBothSidesOfThePair`.
**Issue:** `internal/fabricengine/spawn_test.go:65-67` builds a `Location` under `t.TempDir()` with no repository.
`IsPrimeWorktree` calls `PrimeName` (`internal/fabricengine/worktreelist.go:165-170`), which fails there, and card 7 returns such failures before any spawn.
The request it pins is `TestSpawnStatusPush`'s task-pair row.

### [MEDIUM] TestOriginWrites_LocalMarkedBranch is planned in a package that cannot import hubforge

**Location:** card 5, `plan:internal/fabricengine#TestOriginWrites_LocalMarkedBranch`.
**Issue:** `internal/fabricengine/export_test.go:148-152` records that an internal fabricengine test file cannot import hubforge: hubforge imports fabriccli, which imports fabricengine.
That is why `CheckRemoteBranchRequestForTest` and `ArchiveWeftTipForTest` (`export_test.go:570`) exist.

### [MEDIUM] The --local refusal's way forward fails for its own input

**Location:** card 4, `ErrLocalAddOverOrdinary` and `TestRunCLI_LocalPairScenario`.
**Issue:** `internal/fabricengine/add.go:100-106` refuses `ErrBranchExists` for any existing warp branch, and card 4 keeps that for an unmarked one.
The CLI step over a live ordinary pair reaches the existing-directory refusal at `add.go:108-115` first, under card 4's own ordering.

### [MEDIUM] The pending-flag protocol states no bound

**Location:** card 8, `markPending`, `takePending` and `coalesceFlagged`.
**Issue:** Each flag holds a full `PushSpawn`, and step 4 respawns that request as written through `spawnDetachedPushFn`.
Card 8 introduces the try-lock and the respawn edge, but its Intent states neither what they let through nor what bounds them, as the rubric's attack-surface question requires.
Cards 2, 3, 4, 7 and 9 each state theirs.

### [MEDIUM] Verify skips the tests that read PATTERN.md and docs/

**Location:** the overview's `## verify:` section.
**Issue:** `internal/pattern/overview_test.go:28` checks the real `PATTERN.md`, and `internal/lyxcwd/docslink_test.go` checks the links under `docs/`.
Several cards edit both files, and neither package is in either `go test` line.

### [LOW] Card 12's verbs state no bound

**Location:** card 12, `internal/idecli#Command`.
**Issue:** `seed` and `open` are new verbs, and the card's Intent has no statement of what they let through or what bounds them, unlike card 13's `window`.

### [LOW] The command-key error points at a layer that is never read

**Location:** card 14, `Call` step 3.
**Issue:** The error reuses "correct it, then <Rerun>", but `ExecutableSetting` reads only the template and the board default layer.
The ignored-value warning already names `lyx config set --layer default`.

### [LOW] ErrPushLockBusy misreports after a landed push

**Location:** card 8, the synchronous pushers' acquire inside `coalesceFlagged`.
**Issue:** `internal/fabricengine/pushanchored.go:58`'s text says "neither side was pushed", which is false once a step ran.
`commitRecordsVerb` would then report a failed push.

### [LOW] The local-only gate refusal names an input that never reaches it

**Location:** card 5, `checkRemoteBranchRequest`.
**Issue:** Card 5's reaching input is `Add`'s leftover replacement, but a leftover is a branch present only on origin, so no local mark exists there.
The refusal's reason ends without a next step.

### [LOW] TestLoomPark is missing from card 10

**Location:** card 10, its Edit list.
**Issue:** `internal/loomcli/drivewiring_test.go:27-29` builds `commitStatusDeps{... Commit: func(string) error ...}`, which no longer compiles once card 10 changes the `Commit` shape.

### [LOW] runRowCommand rides on land.go's allowlist entry

**Location:** card 15, `runRowCommand`.
**Issue:** `cmd/lyx/verifiedtree_test.go:26` admits `land.go` with the reason that it runs the landing config's deploy command alone.
Card 15 relies on that entry without updating its reason.

### [NIT] The --window envelope text names the launcher

**Location:** `internal/battencli/cli.go:396`, which card 15 does not cover.
**Issue:** The text says the window "opens through %s at the Open-Window row".
After card 15 the row runs a configured command.

## Rejected

- affected-set: card 7 leaves the commit tests that read the push recorder unlisted. Rejected: they compare `WarpPath` and `WeftPath` field by field (`internal/fabricengine/commit_integration_test.go`, around lines 263 and 290), so new `pushCall` fields break nothing.
- guard-reach: card 8's respawn of an unowed side is unreachable in production. Rejected: D1 prescribes that branch, and `TestCoalesceFlagged_PendingFlags` reaches it.
- guard-reach: card 3's `PushAnchored` local-skip mapping is unreachable after card 10. Rejected: D3 names `PushAnchored` among the pushes that skip a marked branch.
- guard-reach: `ResetPairCode`'s leased remote update may touch origin for a local pair. Rejected as unverified: the fork itself notes that a local pair has no remote branch, and it saw no path that writes one.
- way-forward: the refusals of cards 12 and 13 name no next step. Rejected: `PATTERN-refusal-way-forward` binds webster, shed and loom only, and the decision record's constraint lists no ide or reed refusal.

## Focus notes

Every focus directive was reached:
- Attack surface, cards 7 to 10: F9 is the gap. Cards 7, 9 and 10 state their bounds.
- Reachability, cards 4 to 7: F4, F8 and F14.
- CLI shape, cards 12 and 13: both verbs are operator-audience leaves whose `Use` forms the `PATTERN-cli-cobra` audience bullet admits. F11 covers the missing bound.
- Card 15: its member glyphs need no re-sign arrow, and its tests duplicate no existing test.
- Card order: no card uses a symbol a later card creates.
- Fidelity: D1 to D4 and every constraint are carried. Card 11 is licensed by D4's sentence keeping the quoting beside the shell choice.
- Verify coverage: F10.
