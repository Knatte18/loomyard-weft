---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: handler
    location: _lyx/plan/01-hub-workspace-file.md (Tests, checkPrimeWritesHubWorkspace bullet)
    summary: card 1's checkPrimeWritesHubWorkspace asserts no _portals directory, but earlier TestSpawnScenario steps add pairs through Add, which keeps creating _portals links until card 4, so the check fails from card 1 through card 3
    fix: In card 1's checkPrimeWritesHubWorkspace bullet, replace "and no `_portals` folder or directory" with "and no `_portals` folder in the file". Card 4's TestVerbsRegenerateHubWorkspace already asserts that no verb leaves a `_portals` entry, and card 1's TestWriteHubWorkspace asserts the write creates no `_portals` directory, so no assertion moves to a later card.
  - id: F2
    severity: BLOCKING
    class: design
    origin: handler
    location: _lyx/plan/01-hub-workspace-file.md (Intent, hubWorkspaceFolders item 3)
    summary: hubWorkspaceFolders lists every non-main, non-prunable entry of git worktree list, so a warp worktree registered outside the hub becomes a ../ folder, breaking the constraint that the file references hub-inward folders only
    fix: In card 1's hubWorkspaceFolders item 3, add that an entry whose cleaned worktree path is not a direct child of `l.HubPath` is skipped with a Debug log naming its path, like an entry that fails to resolve. In TestRegenerateHubWorkspace row 5, add a third skip exit, a warp worktree added by `git worktree add` to a directory outside the hub, which is left out while the other pair stays listed.
  - id: F3
    severity: BLOCKING
    class: design
    origin: handler
    location: _lyx/plan/05-retired-artifact-sweep.md (Intent, the emptied _portals chain bullet)
    summary: the sweep removes the emptied _portals directories with a bare removeContainedPath call, skipping the gate's ownership and dirtiness checks that PATTERN-fabric-destruction-chokepoint requires of every executor, and no existing ownership kind admits a _portals path
    fix: In card 5, add `internal/fabricengine#resolvePathOwnership` to `**Edit:**` and `internal/fabricengine#checkPathRequest` to `**Uses:**`. In the chain bullet, state that each emptied directory, and `_portals` itself with the hub as container, first passes `checkPathRequest` with `ownedUnderGeometryRoot` over the `_portals` container and `dirtinessNA("an emptied directory holds no content")`, as removeLaunchers does for its directory, and only then is removed through `removeContainedPath` with `recursive` false. `resolvePathOwnership`'s geometry-root set grows to the `_launchers` and `_portals` containers. Add a sentence to the Intent bounding the set to these two containers and no other root.
  - id: F4
    severity: MEDIUM
    class: scope
    origin: lens:affected-set
    location: internal/vscode/config.go:7
    summary: the vscode package doc still says the prime's settings are spliced into a workspace file under _launchers/, D8 requires every doc comment naming the workspace file to be updated, and no card targets the file
    fix: In card 1, add `internal/vscode/config.go` to `**Edit:**` after `internal/fabricengine/doc.go`, and add a Docs line saying its Limitation paragraph names the hub workspace file at `<hub>/<RepoName>.code-workspace` and the hub directory as what relative settings resolve against. In 00-overview.md's `## verify:`, append `./internal/vscode` to both `go test` lines.
  - id: F5
    severity: MEDIUM
    class: consistency
    origin: lens:way-forward
    location: _lyx/plan/05-retired-artifact-sweep.md (Intent, sweepRetiredHubArtifacts)
    summary: the sweep returns []string and no error, and the card never says what happens when a removeLink, removePath or removeContainedPath call refuses or fails, so a gate refusal can vanish with no detail and no Warn
    fix: "In card 5's sweep bullets, add: every refusal or error from removeLink, removePath or removeContainedPath, except the OS refusing a non-empty directory that still holds a kept entry, becomes one HubDetails entry naming the path and the cause, and never stops the sweep or fails Reconcile. Add a row to TestReconcile_SweepsRetiredHubArtifacts: with `_launchers/<AnchorRel>` replaced by a symlink to a directory outside the hub that holds a `<prime>.code-workspace` file, Reconcile leaves that file in place and HubDetails names the refusal."
  - id: F6
    severity: MEDIUM
    class: consistency
    origin: lens:guard-reach
    location: _lyx/plan/01-hub-workspace-file.md (Tests, TestRegenerateHubWorkspace row 12)
    summary: the concurrent row passes with or without the hub lock, since one goroutine's list-to-write window cannot straddle the other's whole Add and two regenerations, and it calls AddPair, which fails through t.Fatal, from goroutines
    fix: "Rewrite row 12: hold the lock with `lock.AcquireWriteLock(HubWorkspaceLockPath(hub))`, start `RegenerateHubWorkspaceWithinForTest` with a wait longer than the step in a goroutine, create pair `b` with a raw `git worktree add` while the lock is held so no verb regenerates, release the lock, join, and assert the file lists `b`. No pair is added from a goroutine."
  - id: F7
    severity: MEDIUM
    class: consistency
    origin: lens:guard-reach
    location: _lyx/plan/03-reconcile-launchers-restored.md (Intent, mark read failure)
    summary: the mark-read failure is declared a fail-loud wiring guard rejected upstream, but git worktree list uses git's own config parser and readBranch reads only HEAD through go-git, so a config git reads and go-git cannot decode reaches this branch
    fix: In card 3, delete the two sentences that declare the mark-read failure a fail-loud wiring guard with no reaching test. State instead that the failure is a raw read error appended to the pair's Detail, under refusal-spec's transient exemption for a propagated raw I/O error, and that it restores nothing.
  - id: F8
    severity: LOW
    class: consistency
    origin: handler
    location: _lyx/plan/05-retired-artifact-sweep.md (Intent, retired workspace path bullet)
    summary: the retired file is removed through removePath, whose removal is recursive, after an Lstat, so a directory swapped in between is deleted with its contents, breaking D7's bound of never removing a directory
    fix: In card 5's retired-path bullet, replace the removePath call with `checkPathRequest` (same ownership and dirtiness) followed by `removeContainedPath(launchersDir(l), path, false)`, recording `KindPathRemoved` with detail `single`, so the OS refuses a non-empty directory planted after the Lstat.
  - id: F9
    severity: LOW
    class: consistency
    origin: handler
    location: _lyx/plan/01-hub-workspace-file.md (Tests, TestWriteHubWorkspace bullet)
    summary: the new truncated-prefix row in TestWriteHubWorkspace duplicates that test's own changed-rewrite assertion, since writeHubWorkspace only compares bytes, and row 7 of TestRegenerateHubWorkspace already covers a truncated file through the regeneration
    fix: In card 1, delete the sentence "`TestWriteHubWorkspace` gains a row that writes a truncated prefix of the bytes over the file and asserts the next write reports `true` and leaves the whole content."
  - id: F10
    severity: LOW
    class: scope
    origin: lens:affected-set
    location: internal/fabricengine/portallauncher_test.go:1-5
    summary: the test file's header names PortalsDir, PortalLink and portalTarget and pins the _portals layout, card 4 deletes all three, and no card edits the header
    fix: In card 4, add `internal/fabricengine#wantMenuLauncherName`, `internal/fabricengine#TestRemoveLaunchers_PreservesForeignContent`, `internal/fabricengine#TestRemoveLaunchers_DirRemovalIsContained` and `internal/fabricengine#TestRemoveLaunchers_EmptyDirRemovedAndRecorded` to `**Edit:**` after `internal/fabricengine#TestGate_ZeroValueDeclarationsAreRefusals`. Add a line to the Intent saying the `portallauncher_test.go` header names the launcher accessors and the `_launchers` layout only.
  - id: F11
    severity: LOW
    class: consistency
    origin: lens:guard-reach
    location: _lyx/plan/01-hub-workspace-file.md (Tests, checkPrimeWriteFailure and TestWriteHubWorkspace_RefusesEscapingSymlink)
    summary: an escaping symlink at the workspace path fails at root.ReadFile before any write, so both rows reach the read-error branch, never the failed-write branch the card and the acceptance criterion name
    fix: In card 1's checkPrimeWriteFailure bullet, replace the escaping symlink with a file of different bytes at `HubWorkspacePath` whose mode is `0o444`, inside a parent made read-only and restored in cleanup, so `root.WriteFile` fails. Keep the assertions that Spawn returns the write error, CodeLauncher is not called and the file keeps its bytes. Leave TestWriteHubWorkspace_RefusesEscapingSymlink as the read-path refusal and say so in its bullet.
  - id: F12
    severity: LOW
    class: consistency
    origin: lens:guard-reach
    location: _lyx/plan/03-reconcile-launchers-restored.md (Tests, read-only row)
    summary: mode 0o555 on _launchers/<AnchorRel> blocks entry creation only on POSIX as non-root, so the restore-failure row cannot fail on Windows
    fix: In card 3's read-only row, replace the `0o555` arrangement with `_launchers/<AnchorRel>` replaced by a directory link (`fslink.CreateDirLink`) to a directory outside the hub, so `writeLaunchers`' os.Root mkdir refuses on every platform. Assert the same `already_healthy` Action, the `launcher restore failed` Detail and no Error.
  - id: F13
    severity: LOW
    class: consistency
    origin: lens:guard-reach
    location: _lyx/plan/02-verbs-regenerate.md (Tests, row 5)
    summary: row 5 sets SetAddFailAfterWarpHookForTest, a process-global hook only a non-parallel test may set, yet unlike rows 6 and 8 it is not declared serial, so parallel rows' Add calls can hit the hook
    fix: In card 2's row 5, add "This row is serial, because the hook is process-global."
---

### [BLOCKING] Card 1's spawn check asserts a portal-free hub before card 4 retires portal creation

**Location:** `_lyx/plan/01-hub-workspace-file.md` (Tests, `checkPrimeWritesHubWorkspace`)

**Issue:**
`TestSpawnScenario`'s earlier steps call `hubforge.AddPair`, at `internal/ideengine/spawn_clean_integration_test.go:83,117,155,198` and `internal/ideengine/spawn_hub_integration_test.go:117`.
Until card 4, `Topology.add` still calls `createPortal` (`internal/fabricengine/add.go:379`), which creates `_portals/<AnchorRel>/<slug>`.
Card 1's new assertion of "no `_portals` ... directory" therefore fails at cards 1, 2 and 3.
That breaks the spec's rule that a card never depends on what a later card does.

### [BLOCKING] The folder listing admits worktrees outside the hub

**Location:** `_lyx/plan/01-hub-workspace-file.md` (Intent, `hubWorkspaceFolders` item 3)

**Issue:**
`List` returns every registered linked worktree of the warp repository, wherever it lives (`internal/fabricengine/worktreelist.go:28-35`).
`lyxcwd.ResolveWorktree` succeeds for one outside the hub, taking its parent directory as the hub (`internal/lyxcwd/lyxcwd.go:107-126`).
Reconcile already expects such entries: `warpLayoutFor` falls back for a root that is not a direct child of the hub (`internal/fabricengine/warplayout.go:21-26`).
Card 1 then relates the path to the hub, which writes a `../` folder.
That breaks the decision record's `PATTERN-hub-containment` constraint ("the file references hub-inward folders only") and D2's "a code worktree of the hub".

### [BLOCKING] The `_portals` chain removal bypasses the gate's ownership and dirtiness checks

**Location:** `_lyx/plan/05-retired-artifact-sweep.md` (Intent, the emptied `_portals` chain bullet)

**Issue:**
`pattern/PATTERN-fabric-destruction-chokepoint.md` requires every executor to check containment, ownership and dirtiness.
`removeContainedPath` is the raw rooted removal under the executors (`internal/fabricengine/destroy.go:1209`).
Its only use outside `destroy.go`, in `removeLaunchers`, runs `checkPathRequest` first (`internal/fabricengine/launchers.go:300-312`).
Card 5 calls it bare, with no ownership or dirtiness declaration.
No existing ownership kind could be declared either: `ownedUnderGeometryRoot` refuses any root whose base is not `_launchers` (`internal/fabricengine/destroy.go`, the `resolvePathOwnership` `UnderGeometryRoot` case).
This is a violation of a listed constraint, and the fix was checked with `validate-plan --overlay`.

### [MEDIUM] The vscode package doc still places the workspace under `_launchers/`

**Location:** `internal/vscode/config.go:7`

**Issue:**
D8 requires every doc comment naming the hub workspace file to be updated, and D3 moves the relative-settings warning to name the hub directory.
The package doc still says the settings are spliced "into a workspace file under _launchers/".
No card targets the file, and `## verify:` does not run `internal/vscode`, which has untagged tests only.

### [MEDIUM] The sweep's refusal and error path is unspecified

**Location:** `_lyx/plan/05-retired-artifact-sweep.md` (Intent)

**Issue:**
`sweepRetiredHubArtifacts` returns only details, and the card names no outcome for a gate refusal or I/O error from its removals.
Example: an escaping symlink at `_launchers/<AnchorRel>`.
- The plain `Lstat` sees the retired file.
- `removePath` refuses at containment (`internal/fabricengine/destroy.go:908`).
- `retiredHubArtifactsPresent` reads through the hub root, so it sees nothing and logs no Warn.

The refusal disappears, contrary to the chokepoint pattern's "A gate refusal is never silently discarded".

### [MEDIUM] The concurrent-regeneration row cannot detect a missing lock

**Location:** `_lyx/plan/01-hub-workspace-file.md` (row 12)

**Issue:**
Each goroutine's `Add` regenerates through its own deferred call, and each then regenerates again explicitly.
A lost pair would need one listing-to-write window to straddle the other's whole tail, so the row passes without the lock.
`hubforge.AddPairWith` also fails through `tb` (`internal/hubforge/hub.go:417-423`), which is `t.Fatal` called off the test goroutine.

### [MEDIUM] The mark-read failure is mislabelled a wiring guard

**Location:** `_lyx/plan/03-reconcile-launchers-restored.md` (Intent)

**Issue:**
`BranchMarkedLocal` decodes the repository config with go-git (`internal/gitrepo/branchmark.go:47-55`).
The checks upstream of it do not decode that config:
- `git worktree list` uses git's own parser.
- `readBranch` reads only HEAD.

A config git accepts and go-git rejects reaches this branch from on-disk state.
That is not a wiring guard in `refusal-spec.md`'s sense, and the card names no test for it.

### [LOW] The retired-file removal is recursive after an `Lstat`

**Location:** `_lyx/plan/05-retired-artifact-sweep.md` (Intent, retired path bullet)

**Issue:**
`removePath` calls `removeContainedPath` with `recursive` true (`internal/fabricengine/destroy.go:1265`).
A directory planted between the `Lstat` and the removal is deleted with its contents.
That breaks D7's bound: "never a directory".

### [LOW] The truncated-prefix row in `TestWriteHubWorkspace` duplicates existing coverage

**Location:** `_lyx/plan/01-hub-workspace-file.md` (Tests)

**Issue:**
`TestWriteHubWorkspace` already asserts the changed-rewrite case (`internal/fabricengine/hubworkspace_test.go:141-144`).
A truncated prefix is just another set of different bytes for a byte comparison.
Row 7 of `TestRegenerateHubWorkspace` already covers the truncated file through the regeneration.

### [LOW] The `portallauncher_test.go` header goes stale

**Location:** `internal/fabricengine/portallauncher_test.go:1-5`

**Issue:**
The header lists `PortalsDir`, `PortalLink` and `portalTarget` and pins the `_portals` layout, and card 4 deletes all three.
The spec's way forward for a header-only change to a test file is to name the file's surviving members (`loom-plan-spec.md`, check 43).

### [LOW] The spawn failed-write row reaches the read branch

**Location:** `_lyx/plan/01-hub-workspace-file.md` (`checkPrimeWriteFailure`)

**Issue:**
`writeHubWorkspace` reads through the root before it writes, and card 1's own Intent says that read refuses the escaping symlink.
The row therefore never reaches the failed-write return that the acceptance criterion names for the prime spawn.

### [LOW] The read-only restore-failure row cannot fail on Windows

**Location:** `_lyx/plan/03-reconcile-launchers-restored.md` (Tests)

**Issue:**
A directory mode of `0o555` blocks creating entries only under POSIX as non-root.
The repository runs on Windows too, as `internal/fslink`'s junction support shows.

### [LOW] Row 5 sets a process-global hook without being serial

**Location:** `_lyx/plan/02-verbs-regenerate.md` (row 5)

**Issue:**
`addFailAfterWarpHook` may be set only by a non-parallel test (`internal/fabricengine/add.go:26-27`).
Rows 2 and 7 call `Add` in parallel, and with the hook set they would hit its failure branch.

## Rejected

- guard-reach NIT, the sweep's unresolvable-prime skip is unreachable from `Reconcile`: D7 prescribes the skip in so many words, and card 5 already reaches it with a direct unit test, so no change to the plan is warranted.

## Directive verdicts

- Fidelity: D1 through D8 and every Constraint are carried, apart from the gaps in F2, F3 and F4.
  D4's exit rules hold against card 2.
  - Add's regeneration is registered once the pre-flight checks pass, so a rollback regenerates.
  - Remove's is registered after `RemoveRefusal` and `inspectPair` and runs after the teardown lock is released.
  - Checkout's is registered after the original-branch capture, so its rollback regenerates.
  - In each case a pre-flight refusal regenerates nothing.

  Not a defect.
- Card order: F1.
  Every other dependency, including card 2's use of card 1's function and card 5's edit of card 1's `regenerateHubWorkspace`, points at an earlier card.
- Attack surface: card 1 bounds its write to the one rooted file and the `_board/.lyx` directory, and card 5 bounds its sweep to fabric-shaped links and the one retired path.
  F3 and F8 are the gaps between those stated bounds and the mechanism the card prescribes.
- Reachability: F5, F6, F7, F11 and F12.
  The lock-timeout, prime-resolution, settings-read and skip rows are reachable.
- Test economy: F9.
  Card 4's deletions name the cards whose rows take over their coverage.
- Verify coverage: F4.
  Every other targeted package runs on both lines.
- Edit-card glyphs: the re-signed `HubWorkspacePath` and `repairPairWiring` carry their arrows.
  `ReconcileResult` is a struct, which takes no arrow, and the lock-wait parameter belongs to a `Create` handle.
  Not a defect.

The target and verify changes that F3, F4 and F10 propose were checked together with `lyx loom validate-plan --overlay`.
The `{}` baseline and the combined run both returned `ok`.
