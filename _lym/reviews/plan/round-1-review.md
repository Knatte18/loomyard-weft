---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    origin: lens:repo-invariants
    location: _lym/plan/03-machine-name-verb.md (Edit cmd/lym#newRoot); cmd/lym/sandbox_coverage_test.go:25-63
    summary: Mounting the new module machine fails TestSandboxCoverage_AllModulesCoveredOrExcluded, a tier0 guard over cmd/lym, and no card lists a sandbox scenario or an excludedModules entry for it
    fix: In card 3, add `- \`cmd/lym#excludedModules\`` under **Edit:** and one Intent sentence saying the card adds a `machine` entry to excludedModules whose Why reads that the verb writes the per-user machine file and sweeps the Bolt of every registered hub, which no sandbox scenario may do. Also add `cmd/lym/sandbox_coverage_test.go` to the ImpactSummary. The overlay validation passed this fix with no new blocking finding.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:guard-reach
    location: _lym/plan/03-machine-name-verb.md step 4 (sweptHubs) and its test lists; internal/lymcwd/lymcwd.go:73-86
    summary: sweptHubs takes the working directory's hub only when lymcwd.Resolve succeeds, which needs cwd to equal a worktree's anchored directory, so a run from a hub subdirectory or the hub container silently adds no hub, though D6 names running from inside the hub as the way forward; no row reaches the cwd branch or its dedup
    fix: "In card 3 step 4, state the outcome of each lymcwd.Resolve result. ErrNotAGitRepo, and a resolved location whose hub holds no Bolt checkout, add no hub. Any other Resolve error, ErrCwdOutsideAnchor from a worktree subdirectory among them, adds no hub, and the result names the working directory as not resolved to a hub, with the way forward: run the verb from a worktree root of that hub. Add TestNameSet rows for a hub reached only through the working directory, which is checked, and for a working directory inside a registered hub, which is listed once. Add a TestNameSweep row in card 4 asserting that a hub reached only through the working directory is swept."
  - id: F3
    severity: MEDIUM
    class: scope
    origin: handler
    location: _lym/plan/03-machine-name-verb.md Intent; _lym/plan/04-owner-sweep.md doc.go paragraph; decision record D5 Bound paragraphs
    summary: Neither card 3 nor card 4 carries D5's bounds on what the collision check lets through, or that a sweep whose replace set holds a name shared with another machine moves that machine's open entries, nor the way forward D5 names; card 3 states only the locally-checked hub and card 4's doc only the unswept-hub bound
    fix: "Add a bound paragraph to card 3's Intent and have card 3's doc.go state it. The collision check lets through: an owner another machine pushes between the pull and the file write; an owner only origin holds on a hub whose pull failed; a former name another host has since taken; a name a live machine holds with no recorded owner on a swept board; and the hostname rule's name when another host shares it. The replace set still keeps the write off every other name. Extend card 4's doc.go paragraph: a sweep whose replace set holds a shared name moves the other machine's open entries under this machine. The way forward is renaming the machine that holds the shared name as its current name with lym machine name, and each machine then takes back its moved entries with lym adopt."
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:guard-reach
    location: _lym/plan/02-owner-replace-set.md tests paragraph; _lym/plan/04-owner-sweep.md steps 4-5 and the exit rule
    summary: No named test reaches the Lost path for a replace-selected entry; card 2 adds no competing-origin row under Replace, and card 4's per-entry lost report and the non-zero exit it drives are reached by no row, since machinecli exposes no seam for a competing write
    fix: "In card 2, add a subtest ReplaceSetLosesToACompetingOriginWrite beside CompetingWriteLosesToOrigins. Its BeforePush pushes boxed's owner elsewhere from the second clone through pushFromOther and setOwnerOnOther. It then calls SetOwners with Machine neve and Replace [box], and asserts that boxed is in Lost with elsewhere and that origin keeps elsewhere. In card 4, add `- \\`plan:internal/machinecli#nameDeps\\`` and `- \\`plan:internal/machinecli#productionDeps\\`` under **Edit:**. Give nameDeps a field `SetOwners func(boardDir string, req boardengine.OwnerRequest) (boardengine.OwnerWrite, error)`, which productionDeps fills with boardengine.SetOwners and sweepHub calls. Add a TestNameSweep row whose fake returns one Lost entry, asserting the per-entry report with the standing owner, the re-run and lym adopt ways forward, and a non-zero exit. The overlay validation passed the target additions with no new blocking finding."
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:guard-reach
    location: _lym/plan/04-owner-sweep.md step 5 (pushed) and the TestNameSweep rows; internal/fabricengine/bolt.go fetch-failure skip
    summary: No TestNameSweep row reaches the pushed report D6 prescribes, since the unreachable-origin row fails at the fetch and leaves no unpushed commit, so the re-run that pushes an earlier run's commit is never exercised
    fix: In card 4, add a TestNameSweep row that commits the beam owner already set to the new name in one hub's Bolt without pushing it, as an earlier run whose push failed leaves it. The row then runs lym machine name with that name and asserts that the hub is reported pushed, that nothing is recorded, and that origin's head holds the commit.
  - id: F6
    severity: LOW
    class: consistency
    origin: lens:affected-set
    location: _lym/plan/02-owner-replace-set.md doc.go paragraph; internal/boardengine/doc.go:174-175
    summary: "Card 2's doc.go edit names only the Takeover sentence at :183, but its change also falsifies :174, which gives the commit as board: owner <machine> for <slugs>, and :175, which says the commit sets each named beam project and the prime when asked"
    fix: "In card 2's doc.go paragraph, also rewrite :174 so the commit form gains the variant board: owner <machine> replacing <names>, and :175 so the entries the commit sets include the open beam projects and the prime that the replace set selects."
  - id: F7
    severity: LOW
    class: design
    origin: lens:guard-reach
    location: _lym/plan/02-owner-replace-set.md writeOwners bullets; internal/boardengine/ownerset.go:178-184
    summary: The overlap clauses for Replace beside req.Prime or req.Slugs would leave a stale lost entry, because the Prime branch records lost for an owner elsewhere before the replace selection writes the same subject, so SetOwners could report the prime as both recorded and lost
    fix: In card 2's writeOwners bullets, state that a subject the replace selection writes is deleted from lost, so no subject is ever both recorded and lost. Add a row to TestSetOwners' outcomes for primeOwner box with Prime true and Replace [box], which records the prime and reports no Lost entry.
  - id: F8
    severity: LOW
    class: decision
    origin: lens:guard-reach
    location: _lym/plan/03-machine-name-verb.md steps 3 and 5, and the showName paragraph
    summary: Card 3 leaves three error branches without an outcome. ReadOwners can fail in pullOwners. ReadFile can fail to read an existing file, as on a directory or a permission error, which is neither decodes nor does not decode. FilePath can return an error in showName.
    fix: "In card 3, state each outcome. A hub whose ReadOwners fails refuses the set naming the hub and the error, and writes nothing. A machine file that exists but does not read refuses the set with the read error and the file's path, and is not replaced. showName with a FilePath error prints that error and exits non-zero. Add a row for each to TestNameSet or TestNameShow."
  - id: F9
    severity: LOW
    class: design
    origin: handler
    location: _lym/plan/04-owner-sweep.md TestNameSweep row 1
    summary: TestNameSweep row 1 re-asserts at the verb that a done entry, an owner under another name and an owner only origin held are handled, which card 2's ReplaceSetRewritesOpenBeamsAndPrimeUnderAFormerName, ReplaceSetLeavesOtherOwners and ReplaceSetSelectsAnOwnerOnlyOriginHeld pin at SetOwners; the verb's own pull also runs before the sweep, so its origin-only case never reaches the selection under the write's pull
    fix: In card 4, narrow TestNameSweep row 1 to the verb's own behaviour. Every swept hub's open beam projects and prime under the former name read the new name on origin, and the replace set holds the file's names other than the new name. Drop the done-entry, other-name and origin-only clauses.
  - id: F10
    severity: LOW
    class: design
    origin: handler
    location: _lym/plan/04-owner-sweep.md TestNameSweep row 5
    summary: The row the output after a rename carries the recoveries tests the wording the acceptance criterion marks Review, which the code review reading the text meets, so the card plans a test for static text
    fix: In card 4, cut the TestNameSweep row the output after a rename carries the recoveries.
  - id: F11
    severity: LOW
    class: consistency
    origin: lens:guard-reach
    location: _lym/plan/03-machine-name-verb.md step 6
    summary: "The collision refusal's hand-write way forward is conditional on a repair back to a lost name, a state checkCollision cannot see, since its signature takes only the name, this machine's names and the hubs' owners"
    fix: "In card 3 step 6, make the refusal always name both ways forward: choose another name, or, when the name was this machine's and its names were lost, write {\"name\": \"<name>\"} to the file's path by hand. Have the neve row of TestNameSet assert that both appear."
  - id: F12
    severity: NIT
    class: consistency
    origin: handler
    location: _lym/plan/03-machine-name-verb.md TestNameSet rows
    summary: The row expecting names osl-1046 then osl46 holds only when the Hostname seam answers osl-1046, since rewriteFile appends the hostname rule's name when absent, and the card states no hostname for TestNameSet
    fix: In card 3, state that TestNameSet hands in the hostname osl-1046 through the Hostname seam, as cli_test.go hands in a fixed hostname.
---

### [BLOCKING] The new module fails the sandbox coverage guard

**Location:** `_lym/plan/03-machine-name-verb.md`, Edit `cmd/lym#newRoot`; `cmd/lym/sandbox_coverage_test.go:25-63`
**Issue:** `TestSandboxCoverage_AllModulesCoveredOrExcluded` is a tier0 `//lym:guard` test that walks `newRoot()`. Every registered module needs either a `**Covers:**` scenario in a `tools/sandbox/*SUITE.md` file or an `excludedModules` entry with a reason. Card 3 mounts `machine` and lists neither, so its own card gate fails. An exclusion fits better than a scenario, because the verb writes the per-user machine file and sweeps real hubs' Bolts, as the existing exclusions of `hibernate` and `slack` reason. The `validate-plan --overlay` run over the added target reported no new blocking finding.

### [MEDIUM] The working directory's hub is lost outside a worktree anchor

**Location:** `_lym/plan/03-machine-name-verb.md` step 4; `internal/lymcwd/lymcwd.go:73-86`
**Issue:** `lymcwd.Resolve` requires cwd to equal a worktree's anchored directory. It answers `ErrCwdOutsideAnchor` from a subdirectory and `ErrNotAGitRepo` from the hub container. The card files every failure under "a directory in no hub adds none". A run from inside an unregistered hub, anywhere but at a worktree root, therefore sweeps and checks nothing and says nothing, yet D6's bound gives "running the verb from inside it" as the way forward. No row covers the cwd branch or the dedup with a registered hub.

### [MEDIUM] D5's bounds on the collision check and the shared-name sweep are not carried

**Location:** card 3 Intent; card 4's `doc.go` paragraph; decision record D5 "Bound:" paragraphs
**Issue:** D5 states the following bounds:
- what the collision check lets through: owners pushed between the pull and the write, origin-only owners on a hub whose pull failed, a former name another host took, a live machine's name with no recorded owner, and a shared hostname name;
- that a sweep over a shared name moves the other machine's open entries;
- the way forward: rename the machine holding the shared name, then take entries back with `lym adopt`.

Card 3 states only the locally checked hub, and card 4's doc only the unswept hub. The attack-surface rule requires the card that introduces the verb to state what it lets through and what bounds it.

### [MEDIUM] No test reaches the Lost path of a replace-selected entry

**Location:** card 2's tests paragraph; card 4 steps 4–5 and the exit rule
**Issue:** `Lost` comes from the post-push re-read over `written` (`ownerset.go:141-160`). `CompetingWriteLosesToOrigins` exercises it only for `Slugs`. Card 2 adds no competing-write row under `Replace`. Card 4 prescribes a per-entry lost report and a non-zero exit, but no row reaches either: a competing write needs `BeforePush`, which the verb never sets, and `nameDeps` has no owner-write seam. The overlay run over the added `nameDeps` and `productionDeps` targets reported no new blocking finding.

### [MEDIUM] No test reaches the pushed report

**Location:** card 4 step 5 and the `TestNameSweep` rows
**Issue:** D6 prescribes reporting a hub as pushed when a re-run only pushes an earlier run's commit. The unreachable-origin row fails at the fetch, a skip that leaves no commit (`fabricengine/bolt.go`, `BoltSkipFetchFailed`), so no row produces an ahead Bolt for the re-run to push.

### [LOW] Card 2 misses two doc.go sentences its change falsifies

**Location:** card 2's `doc.go` paragraph; `internal/boardengine/doc.go:174-175`
**Issue:** Line 174 gives the commit as `"board: owner <machine> for <slugs>"`, and card 2 adds the `replacing <names>` form. Line 175 says the commit sets each named beam project and the prime when asked, but a replace-selected entry is neither named nor asked for.

### [LOW] Replace beside Prime can report the prime both recorded and lost

**Location:** card 2's `writeOwners` bullets; `internal/boardengine/ownerset.go:178-184`
**Issue:** The Prime branch writes `lost[prime]` when the prime owner reads as another machine. The card's replace selection then writes the prime "once even when req.Prime also asked for it", but the `lost` entry stays. The post-push loop then records the prime while `Lost` still holds it. `SetOwners` is a shared API, so the combination the card promises to handle has to come out consistent.

### [LOW] Three error branches of card 3 have no stated outcome

**Location:** card 3 steps 3 and 5, and the `showName` paragraph
**Issue:**
- `ReadOwners` returns an error on an unreadable `board.json`.
- `machinename.ReadFile` returns `(File{}, false, err)` for an existing file that does not read, which differs from a decode error and fits neither of `rewriteFile`'s cases.
- `FilePath` can return an error while `showName` runs.

The card states no outcome for any of the three.

### [LOW] TestNameSweep row 1 repeats card 2's SetOwners rows

**Location:** card 4, `TestNameSweep` row 1
**Issue:** Done-entry skipping, other-name owners and origin-only selection are `SetOwners` behaviour that card 2's rows pin. At the verb, the origin-only case does not even reach the selection under the write's pull, because `pullOwners` has already pulled. The verb's own behaviour is which hubs it sweeps and the replace set it builds.

### [LOW] A test row asserts the Review-marked recoveries text

**Location:** card 4, `TestNameSweep` row 5
**Issue:** The acceptance criterion on the recoveries output opens with `Review:`. Under the rubric, a criterion about wording is met by the code review reading the text, and a card that plans a test for it is a finding whose fix cuts the test.

### [LOW] The collision refusal's way forward depends on state checkCollision cannot see

**Location:** card 3 step 6
**Issue:** `checkCollision(name, mine, hubs)` cannot tell that it faces a repair back to a lost name, so the conditional "for a repair back to a lost name it names writing …" is unreachable as written. No row asserts the hand-write way forward.

### [NIT] TestNameSet's hostname is unstated

**Location:** card 3, `TestNameSet` rows
**Issue:** `rewriteFile` appends the hostname rule's name when it is absent. The expected `["osl-1046","osl46"]` therefore holds only when the seam's hostname is `osl-1046`, and the card leaves that unstated.

## Focus verdicts

- Granularity, card 3: one card holds the one CLI surface and every private helper of it, so the separate Create glyphs are no granularity defect. Only `Command`, `RunCLI` and `RunCLIIn` are reused outside the package.
- Card 2's glyphs and the re-sign arrow: no listed member's parameters, results or receiver change. `OwnerRequest` gains a field, which takes no arrow, so the glyphs fit.
- Card 2's `PATTERN.md` Edit target is justified: `pattern-line-unprescribed` needs a card that edits `PATTERN.md` to fence the declared `changes` line, even when its text is unchanged. The `pattern/PATTERN-owner-machine.md` bullet rewrite changes existing text, so `Edit` is correct.
- Attack surface, card 2: its Intent states the bound (recorded owners in the caller's set; open beams and the prime only; never a done entry or an empty owner). For cards 3 and 4, see F3.
- Card order and research steps: card 1 precedes card 3's use of `HostnameName`, and card 4 edits only symbols card 3 creates. No card leaves a design fact to be found while it runs, and no card edits this task's board entry, which `lym board get` answers as absent.
- Every ImpactSummary carries a blast-radius conclusion. Card 4's "no other package changes" is accurate, since it only calls `boardengine.SetOwners`.
- Test economy, card 3 against `internal/machinename`: `TestNameShow` and `TestNameRefusesWithoutWriting` assert the verb's output and its no-write guarantee, which no `machinename` test covers. Card 3's unreachable-origin row (collision checked locally only) and card 4's (sweep failed) pin different behaviour.
- Test economy, card 2: the new rows and subtest cover replace selection, which no existing row reaches.

## Rejected

- repo-invariants NIT on `ownerset.go`'s file header: the header stays true for named projects. A header edit would need the file self glyph beside the card's member glyphs, which `redundant-file-target` refuses.
