---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: _lym/plan/03-machine-name-show.md:14
    summary: Card 3 mounts the machine module on newRoot but adds neither a sandbox Covers tag nor an excludedModules entry, so the tier0 TestSandboxCoverage_AllModulesCoveredOrExcluded fails in card 3's own gate
    fix: In card 3 add `cmd/lym#excludedModules` under **Edit:** and an Intent line saying excludedModules gains `machine` with a reason such as that the verb writes the per-user machine file and pushes every live hub's Bolt, so internal/machinecli's own seam and hubforge tests exercise it (overlay-checked against validate-plan, no new blocking finding)
    origin: lens:repo-invariants
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lym/plan/01-owner-replace-set.md:26
    summary: The commit subject for the only request shape card 4 sends (ReplaceOwners with no Slugs and no Prime) is unspecified, the example needs Prime true, the ReplaceSet row names no wantSubject although the outcome loop requires origin head at the seed when wantSubject is empty, and doc.go's subject sentence goes stale
    fix: "In card 1's Intent pin the subject as `board: owner <machine> for <named subjects>, replacing <non-empty set entries joined by comma and space>`, with `for <subjects>, ` omitted when no slug or prime is named (`board: owner neve replacing box`); replace the `for prime, replacing box` example with one whose request sets Prime; give the ReplaceSetRewritesOwnersRecordedUnderAFormerName row wantSubject `board: owner neve replacing box`; and extend the doc.go sentence to rewrite the Owners section's `board: owner <machine> for <slugs>` and `sets each named beam project's owner` lines (internal/boardengine/doc.go:174-175) for the replace set"
    origin: lens:affected-set
  - id: F3
    severity: MEDIUM
    class: consistency
    location: _lym/plan/04-machine-name-set.md:84
    summary: Three refusals setName prescribes have no reaching TestSetName row, namely the relative config variable (step 2), the registry read error (step 4) and the readOwners error (step 5)
    fix: In card 4's TestSetName row list add a row per refusal through its seam, asserting the envelope's error and way forward and that the file and every fake board stay unchanged - filePath returning a relative-variable error, registryHubs returning an error (the refusal names the record's path), and readOwners returning an error for one hub (the refusal names the hub)
    origin: lens:guard-reach
  - id: F4
    severity: MEDIUM
    class: consistency
    location: _lym/plan/04-machine-name-set.md:82
    summary: D5's repair paths and the output flags for a replaced file and dropped empty entries are reached only by TestNextFile, a per-helper test of a private function, so no verb-level row asserts the acceptance criteria that the output says so or that a hand-written name-only file sweeps owners under the hostname rule's name
    fix: In card 4 drop the `plan:internal/machinecli#TestNextFile` Create glyph and its paragraph, and move each of its cases into TestSetName as a row run through RunCLI that asserts the written file's bytes and the envelope - the osl-1046 to osl46 rename, a file with an invalid name, an undecodable file reported as replaced, a name-only file adding the hostname name and sweeping a fake board's owner recorded under it, an empty entry dropped and reported, and a re-run with the name set leaving the file's bytes unchanged (overlay-checked, no new blocking finding)
    origin: lens:guard-reach
  - id: F5
    severity: MEDIUM
    class: design
    location: _lym/plan/04-machine-name-set.md:45
    summary: cwdHub maps every lymcwd.Resolve failure to ok false, and Resolve refuses any cwd but the exact anchor, so a run from a worktree subdirectory silently skips the hub the working directory lies in, contrary to D6
    fix: In card 4 make the field `cwdHub func(cwd string) (hub string, ok bool, err error)`; lymcwd.ErrNotAGitRepo reads as no hub, lymcwd.ErrCwdOutsideAnchor is refused before any write naming the worktree's anchor directory as the place to run from (D6's way forward of running the verb from inside the hub), and any other error is refused naming cwd; add a TestSetName row whose cwdHub returns ErrCwdOutsideAnchor and asserts the refusal with the file and fake boards unchanged
    origin: lens:guard-reach
  - id: F6
    severity: MEDIUM
    class: consistency
    location: _lym/plan/04-machine-name-set.md:94
    summary: TestSetName_SweepsRealHubs hands in only registryHubs and filePath, so hostnameName reads the developer's real hostname into names and the replace set and cwdHub resolves the test process's directory, making the outcome host-dependent against D8
    fix: In card 4's TestSetName_SweepsRealHubs paragraph state that the test also hands in a fixed hostnameName returning a name no fixture board records and a cwdHub returning ok false, and runs RunCLIIn with a temporary-directory cwd
    origin: lens:guard-reach
  - id: F7
    severity: LOW
    class: consistency
    location: _lym/plan/04-machine-name-set.md:64
    summary: The pushed report needs the Bolt ahead before the write and level after it, but sweepHub reads hasUnpushed only before setOwners
    fix: In card 4 step 8 state that sweepHub reads hasUnpushed before setOwners and again after a setOwners that returned no error and recorded nothing, and reports pushed when the first read was ahead and the second level
    origin: lens:guard-reach
  - id: F8
    severity: LOW
    class: consistency
    location: _lym/plan/03-machine-name-show.md:14
    summary: cmd/lym/seamsignature_test.go pins every module's RunCLI and every RunCLIIn that reads its cwd, and card 3 creates both machinecli seams, card 4's RunCLIIn reading cwd, without adding them to either list
    fix: In card 3 add `cmd/lym/seamsignature_test.go` under **Edit:** with an Intent line adding machinecli.RunCLI and machinecli.RunCLIIn to its two lists (overlay-checked, no new blocking finding)
    origin: lens:affected-set
  - id: F9
    severity: LOW
    class: consistency
    location: _lym/plan/03-machine-name-show.md:1
    summary: docs/overview.md's package tree lists every internal package beside internal/machinename and internal/hibernatecli, and no card adds the new internal/machinecli line
    fix: In card 3 add a **Prosa:** group with `docs/overview.md` and an Intent line giving the package tree's `internal/machinecli/` line, the `lym machine name` verb that shows and sets the machine name and sweeps owners across a rename (overlay-checked, no new blocking finding)
    origin: lens:affected-set
  - id: F10
    severity: LOW
    class: design
    location: _lym/plan/01-owner-replace-set.md:24
    summary: The rule that an entry both Slugs and the replace set name is written once guards a request shape no caller sends and no row reaches
    fix: In card 1 replace the sentence with a fail-loud wiring guard - SetOwners refuses a request carrying ReplaceOwners together with Slugs or Claim, since the rename sweep sends the set alone - or drop the sentence; either way add no test row for it
    origin: lens:guard-reach
  - id: F11
    severity: LOW
    class: consistency
    location: _lym/plan/03-machine-name-show.md:42
    summary: showName's output with no file or no config directory while hostnameName returns an error is unprescribed and no TestShowName row reaches it
    fix: In card 3 add a showName bullet - with no file or no directory and a hostname that yields no name, it prints hostnameName's error, which names its way forward, through output.ErrFields with the path when one resolves, exiting non-zero - and a TestShowName row with a failing hostnameName
    origin: lens:guard-reach
  - id: F12
    severity: NIT
    class: consistency
    location: _lym/plan/01-owner-replace-set.md:30
    summary: The ImpactSummary says no existing row asserts on the new fixture entries, but BeamAndOwnerLandInOneCommit asserts the colours every other open beam holds, which kept would join if seeded with a colour
    fix: In card 1's newOwnerFixture sentence state that closed and kept record no beam colour
    origin: lens:affected-set
  - id: F13
    severity: NIT
    class: consistency
    location: _lym/plan/03-machine-name-show.md:52
    summary: The PATTERN-cli-cobra naming deviation card 3 adds lists machinecli's callees without standalonestate, whose UserStateDir card 4's registryHubs calls
    fix: In card 3 add `standalonestate` to the list of packages the machinecli deviation names
    origin: handler
---

### [BLOCKING] Card 3 breaks the sandbox coverage guard

**Location:** `_lym/plan/03-machine-name-show.md:14`, against `cmd/lym/sandbox_coverage_test.go:76-127`.
**Issue:** `TestSandboxCoverage_AllModulesCoveredOrExcluded` (tier0) collects every module name on `newRoot()`.
It fails on any name that has neither a `**Covers:**` tag in `tools/sandbox/*SUITE.md` nor an `excludedModules` entry.
Its `//lym:watch` lines name `cmd/lym/main.go` and `internal/*cli/*.go`, so card 3's own gate runs it and fails once `machine` is mounted.
No card targets the file.

### [MEDIUM] The replace-only commit subject is unpinned

**Location:** `_lym/plan/01-owner-replace-set.md:26`, `:32`.
**Issue:** `SetOwners` builds `board: owner %s for %s` from `Slugs` and `Prime` (`internal/boardengine/ownerset.go:101-105`).
Card 4 sends `ReplaceOwners` alone, so the subject comes out as `board: owner neve for ` followed by whatever card 1 appends, and the card does not say what that is.
The example `board: owner neve for prime, replacing box` assumes `Prime: true`, which no planned caller sends.
The `ReplaceSetRewritesOwnersRecordedUnderAFormerName` row names no `wantSubject`, but the outcome loop takes an empty `wantSubject` to mean that origin's head stays at the seed (`ownerset_integration_test.go:292-297`), so the row as planned fails.
The set `["box", ""]` also joins to a trailing empty item.
`doc.go:174-175` documents the subject and "each named beam project", and card 1's doc edit leaves both lines unchanged.

### [MEDIUM] Three setName refusals have no reaching row

**Location:** `_lym/plan/04-machine-name-set.md:54`, `:56`, `:58`, against the row list at `:84-92`.
**Issue:** The relative variable, the registry read error and the `readOwners` error are each prescribed refusals.
Each is reachable through a seam card 4 creates, and no `TestSetName` row names an input that reaches it.
`TestShowName`'s relative-variable row covers only the show path.

### [MEDIUM] D5's repair paths are tested only below the verb

**Location:** `_lym/plan/04-machine-name-set.md:82`.
**Issue:** `TestNextFile` calls the private `nextFile` directly.
The acceptance criteria are about the verb: the output reports a dropped empty entry and a replaced undecodable file, and a name-only file sweeps owners recorded under the hostname rule's name.
No `TestSetName` row asserts any of these through `RunCLI`.
A per-helper test also breaks once `setName`'s internals change.
Moving the rows into `TestSetName` covers the same cases at the public surface, and the separate helper test is then redundant.
The two tests do not overlap on the invalid-name case: `TestSetName`'s invalid name is the argument, and `TestNextFile`'s is the file's.

### [MEDIUM] cwdHub drops the working directory's hub off its anchor

**Location:** `_lym/plan/04-machine-name-set.md:45`.
**Issue:** `lymcwd.Resolve` requires cwd to equal the anchored directory exactly (`internal/lymcwd/lymcwd.go:76-84`), and fails with `ErrCwdOutsideAnchor` otherwise (`internal/lymcwd/anchor.go:89`).
`cwdHub` reports that failure as ok false, so a run from a worktree subdirectory reads as a run outside any hub and sweeps only the registry's hubs, with nothing in the output saying so.
D6 sweeps "the hub of the working directory when it lies in one".

### [MEDIUM] The tier1 verb test reads the host's hostname and cwd

**Location:** `_lym/plan/04-machine-name-set.md:94`.
**Issue:** With `hostnameName` at production, the developer's hostname enters `names` and the replace set.
With `cwdHub` at production and `RunCLI`'s empty cwd, the test resolves the process directory as a hub.
In both cases the outcome depends on the machine running the test, and D8 requires every test to sweep only hubs under its temporary directory.

### [LOW] pushed needs a second hasUnpushed read

**Location:** `_lym/plan/04-machine-name-set.md:64`, `:68`.
**Issue:** Step 8 reads `hasUnpushed` only before `setOwners`.
The `pushed` condition, and the `TestSetName` row at `:91`, also need the state after the write.

### [LOW] seamsignature_test.go misses machinecli

**Location:** `_lym/plan/03-machine-name-show.md:14`, against `cmd/lym/seamsignature_test.go:1-3`, `:54-56`.
**Issue:** The file pins every module's `RunCLI`, and every `RunCLIIn` that reads its cwd.
`machinecli`'s `RunCLIIn` reads cwd for `cwdHub`, and no card adds either seam to the lists.

### [LOW] docs/overview.md's package tree lacks machinecli

**Location:** `_lym/plan/03-machine-name-show.md:1`, against `docs/overview.md:362`, `:384`.
**Issue:** The package tree describes every internal package, including `internal/machinename` and `internal/hibernatecli`.
Card 3 creates `internal/machinecli`, and no card adds its line.

### [LOW] The written-once rule guards an unsent request shape

**Location:** `_lym/plan/01-owner-replace-set.md:24`.
**Issue:** The rule covers an entry that both `Slugs` and the replace set name.
Card 4 sends `ReplaceOwners` and nothing else, and no row names an input that reaches the rule.

### [LOW] showName with a failing hostname is unprescribed

**Location:** `_lym/plan/03-machine-name-show.md:42`.
**Issue:** With no file and a hostname that yields no name, `showName` has no name to print, and the card does not say what it prints instead.

### [NIT] The ImpactSummary misstates the new fixture entries' reach

**Location:** `_lym/plan/01-owner-replace-set.md:30`, `:44`.
**Issue:** `BeamAndOwnerLandInOneCommit` asserts the colour picker sees `[#7d2d6b]` only (`ownerset_integration_test.go:437`).
`kept` is an open beam, so a colour seeded on it breaks that row.

### [NIT] The cli-cobra deviation omits standalonestate

**Location:** `_lym/plan/03-machine-name-show.md:52`.
**Issue:** Card 4's `registryHubs` reads `standalonestate.UserStateDir()`, and the deviation line card 3 adds does not name `standalonestate`.

## Focus settlements

- Constraints: `pattern/PATTERN-leaf-packages.md:95-99` already admits `internal/standalonestate` for `internal/machinename`, and `pattern/PATTERN-owner-machine.md:40-42` already has the machine file read before the hostname. Commit `0604f2ea` landed both, so neither needs a card.
  Card 1 carries the third clause, the replace set as the second takeover case. Not a defect.
- D5 and D6 fidelity: every repair path, refusal and report has a card 4 step. The rows that reach them are F3, F4 and F7.
- Scope: `checked_locally_only`, the dropped-entry and replaced-file output fields, and `recoveries` are licensed by D5's "the output names it as checked locally only" and "the output says so", by D6's recovery list, and by the acceptance criteria.
- Granularity: card 4's private types and helpers sit in the one card of the surface they serve, which is what the rubric asks.
  The card is one reviewable unit, the set path of one verb. Not a defect.
- Edit glyphs: card 4's `Command`, `deps` and `productionDeps`, and card 1's `SetOwners`, `writeOwners` and `newOwnerFixture`, keep their signatures, so no re-sign arrow is due. `deps` is a struct type.
- Attack surface, card 1: the Intent states the bound. Only a non-done beam project or the prime whose recorded owner is in the set is selected, and an empty entry matches nothing.
  Card 4 passes every non-empty `names` entry other than the new name. That includes the hostname rule's name, which D5 licenses explicitly along with its shared-hostname bound.
- PATTERN.md on card 1: the writer stencil (`loom-block-plan-format.md:30`) requires a declared `changes` entry to be prescribed verbatim on a card that edits `PATTERN.md`, so the target stands.
  `pattern/PATTERN-owner-machine.md` is correctly an `Edit`, since the card rewrites an existing bullet.
- Research steps and board edits: none. The Bolt directory, the `ApplySkipEnv` fold and the `Mutations` recorder follow `SetOwners`' own shape (`ownerset.go:88-112`).
  `hubregistry.Live` returns `[]Entry` with `HubPath` (`internal/hubregistry/hubregistry.go:98`). No card writes this task's board entry.

## Rejected

- repo-invariants 2 (name the tier1 test's file): the writer stencil says a new test is a `plan:` handle on the package so the plan does not pin its file (`loom-block-plan-cards.md:24`). The implementer splits the files to satisfy the tier scan.
- guard-reach 3 (a machine file that does not read is unprescribed): `ReadFile`'s I/O error is a propagated raw I/O error, which `refusal-spec.md` exempts. The obvious behaviour is to return it unchanged.
- guard-reach 6 (no row for a replace-selected entry in `Lost`): the selection only joins `written`. The re-read that sorts `written` into `Recorded` and `Lost` is shared with named slugs, and `CompetingWriteLosesToOrigins` already covers it. A new row would restate that test.
- Handler: `TestHostnameName` is a new function where the added entry says "a test row". No existing test covers `HostnameName`'s wiring to `os.Hostname`, and no table hands in the real hostname, so a new function is not redundant.
- Handler: `TestSetName_SweepsRealHubs` against card 1's `ReplaceSetSweepsAnOwnerOnlyOriginHolds`. They work at different layers: the verb test alone drives `productionDeps`' real pull, read and owner write, and the collision check against an owner that only origin holds.
