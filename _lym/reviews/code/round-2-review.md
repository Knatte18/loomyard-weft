---
verdict: BLOCKING
findings:
  - id: F26
    severity: BLOCKING
    class: design
    location: internal/dobbyengine/reinterpret.go:159
    summary: settingPlaces finds the hub's worktrees by listing the hub dir for a .git entry and derives each anchor as the prime's offset joined onto it, outside lymcwd and against dobbyengine's told geometry, so weft siblings count as worktrees and a pair with its own anchor is read and written at the wrong dir.
    fix: "Stop dobbyengine from discovering worktrees or anchors. In internal/configreg/configreg.go, give the rewrite the hub's code worktrees as told geometry, for example a new Reinterpretation field or a widened Rewrite signature taking a list of {name, anchor} places. In internal/hubreconcile, fill that list from the code worktrees the walk already holds (fabricengine.CodeWorktree's Path and Anchor, as reconcileHubWide's prime is), or through lymcwd.NamedWorktree(hubPath, name) in the wiring, and pass boardDir for the board instead of reinterpret.go:226 deriving it from lymdirs.BoltDirName. Delete settingPlaces' os.ReadDir, .git probe and offset join at internal/dobbyengine/reinterpret.go:134-161, and drop the lymdirs import and the told-geometry exception added to seam_enforcement_test.go. Have flagRetiredValues and carryLegacyBoardSettings iterate the told places. Add a TestReinterpretPerTaskSettings_HubWorktrees case with a worktree whose anchor differs from the prime's and a weft sibling holding .git, asserting the flag and carry land at the told anchor and the sibling is not read."
  - id: F2
    severity: BLOCKING
    class: design
    carries: F2
    location: internal/dobbyengine/config.go:73
    summary: The beam module name is still written as the literal "beam" at dobbyengine.ConfigSpec's Name, the configreg beam registration and beamcli mergeParent's run_settings key, beside beamsetting.Module that declares it.
    fix: "Replace the literal \"beam\" with beamsetting.Module at internal/dobbyengine/config.go:73 (Spec Name), internal/configreg/configreg.go:146 (the beam Module's Name) and internal/beamcli/mergeparent.go:85 (the run_settings map key of the merge task). Leave the board's retired entry keys in boardengine/store.go as their own constants."
  - id: F27
    severity: BLOCKING
    class: design
    origin: assist:assist
    location: internal/configreg/configreg_test.go:27
    summary: The redundancy report lists TestLookup_BeamCarriesThePerTaskSettingsReinterpretation as contained by TestPrepareRunSettings, which never asserts the beam module's Reinterpretations registration, and the test carries no keep marker.
    fix: "Add the line //testtiming:keep pins the beam module's registration of the beam-per-task-settings reinterpretation, with a Rewrite and RunsWithoutFile set, which TestPrepareRunSettings never asserts directly above func TestLookup_BeamCarriesThePerTaskSettingsReinterpretation at internal/configreg/configreg_test.go:27, after its doc comment. Keep the test."
  - id: F28
    severity: BLOCKING
    class: design
    origin: assist:assist
    location: internal/configreg/runsettings_test.go:77
    summary: The row a beam pull_request merge-in and a land_notice on pass duplicates the row at line 207 of the same TestPrepareRunSettings table, with the same settings and the same wantDocs.
    fix: Delete the row named a beam pull_request merge-in and a land_notice on pass at internal/configreg/runsettings_test.go:77-81. Keep the row at line 207.
  - id: F5
    severity: MEDIUM
    class: design
    carries: F5
    location: internal/dobbyengine/reinterpret.go:222
    summary: The carry into seeded runs misses two cases. A board save by the new binary before the walk drops the retired keys LegacyBeamSettings reads, since board verbs run no hub reconcile. The carry also runs once per hub behind a marker committed to the board's _lym/config, yet writes the untracked .lym local layer of this machine's worktrees only. A seeded run on another machine, or behind an early save, loses its merge-in or its notice.
    fix: "Make the carry reach every seeded run once on each machine that holds one. Run it from the per-machine part of the reconcile, gated by this machine's untracked build stamp or a marker under the machine's .lym, not the hub's committed reinterpretation marker. Take its source from what survives a save: each live worktree's board entry's run_settings beam keys, which the fold writes, plus the retired keys while they remain. Write a key only where the worktree's own files set none, as now. Add a test that saves an unrelated board edit before the carry and asserts a seeded merge-parent task still gets pull_request merge-in and land_notice on in its local layer."
  - id: F12
    severity: MEDIUM
    class: decision
    carries: F12
    location: internal/dobbyengine/landnotice.go:165
    summary: A beam member under a recorded pull request gets its landed line even when its land_notice reads off, though the board entry says a value set lower down always applies, and no added decision licenses the exception.
    fix: Leave the code in place and defer this finding to the run's parent, who records the exception with lym loom decision add or refuses it. If the parent refuses it, change rememberRuns at internal/dobbyengine/landnotice.go:165 to send a member's line only when note.Requested is set, keeping the follow-up on that line.
  - id: F13
    severity: MEDIUM
    class: design
    carries: F13
    location: internal/boardengine/store.go:297
    summary: A project's notify_on_land is dropped by the fold on load and lives only in the raw key, so a board save before the reconcile carry loses a beam project's notice request with nothing to recover it from.
    fix: "Keep a project's legacy notice where a save does not erase it until the carry has run. One way is to keep the project's notify_on_land in Unknown instead of deleting it at internal/boardengine/store.go:297, so the carry still reads it, and have the carry flag or set it. Another is to leave it on the entry under a retained key the carry reads. Correct the foldLegacyBeamFields comment and the boardengine doc.go paragraph to match. Add a test where a beam project carries notify_on_land true, an unrelated board save runs, and the carry still sets land_notice on in the beam worktree."
prior:
  - id: F1
    status: resolved
  - id: F2
    status: open
  - id: F3
    status: resolved
  - id: F4
    status: resolved
  - id: F5
    status: open
  - id: F6
    status: resolved
    answer: The walk flags each worktree's retired values and the changed meaning of a worktree file's pull_request; how it finds those worktrees is F26.
  - id: F7
    status: resolved
  - id: F8
    status: resolved
  - id: F9
    status: resolved
  - id: F10
    status: resolved
  - id: F11
    status: resolved
    answer: TestLookup_BeamCarriesThePerTaskSettingsReinterpretation pins the registration; its missing keep marker is F27.
  - id: F12
    status: open
    answer: No entry was added to the decision record, so nothing settles the held exception.
  - id: F13
    status: open
  - id: F14
    status: resolved
  - id: F15
    status: resolved
  - id: F16
    status: resolved
  - id: F17
    status: resolved
  - id: F18
    status: resolved
  - id: F19
    status: resolved
  - id: F20
    status: resolved
  - id: F21
    status: resolved
  - id: F22
    status: resolved
  - id: F23
    status: resolved
  - id: F24
    status: resolved
  - id: F25
    status: resolved
---

### [BLOCKING] The reinterpretation discovers worktrees and anchors itself

**Location:** internal/dobbyengine/reinterpret.go:159

**Issue:** `settingPlaces` reads `filepath.Dir(worktreeRoot)` with `os.ReadDir`, takes every directory holding a `.git` entry as a worktree, and sets each one's anchor to `filepath.Join(hub, name, offset)`, where `offset` is the prime's anchor.
`carryLegacyBoardSettings` also derives the board as `filepath.Join(filepath.Dir(worktreeRoot), lymdirs.BoltDirName)`.
PATTERN-cwd-resolution says code outside `lymcwd` gets another worktree's `Location` through `lymcwd.NamedWorktree`, that a worktree's anchor comes from its own `.lym-anchor`, and that `dobbyengine`, a bound package, derives no path of its own.
The diff admits the breach in the comment it adds to `seam_enforcement_test.go`.
In this hub, each pair's `<slug>-weft` records sibling holds `.git`, `.lym` and `_lym`, so the walk reads it as a worktree.
A pair whose recorded anchor differs from the prime's is flagged and carried at a dir the runtime lookup, which goes through `lymcwd.NamedWorktree` in `hubgeom.TaskSettingChain`, never reads.
The hub walk already holds each code worktree's path and anchor.
The assistant reviewer found the same defect.

### [BLOCKING] The beam module name is still repeated beside beamsetting.Module

**Location:** internal/dobbyengine/config.go:73

**Issue:** Round 1's F2 asked for the beam module name to be declared once and referenced everywhere.
`beamsetting.Module` now declares it, and boardengine's fold uses it.
`ConfigSpec` still names the spec `"beam"`, the changed configreg registration row names the module `"beam"`, and `mergeParent` keys its run_settings with `"beam"` at `internal/beamcli/mergeparent.go:85`.
PATTERN-code-conventions requires one named constant for a name that can go stale.

### [BLOCKING] The registration test needs its keep marker

**Location:** internal/configreg/configreg_test.go:27

**Issue:** The redundancy report lists this test as contained by `TestPrepareRunSettings`.
That test checks prepared documents and refusals.
It never asserts that `Lookup("beam").Reinterpretations` holds the change with a `Rewrite` and `RunsWithoutFile` set.
The rubric asks for a kept candidate that pins what its covering test cannot see to be marked `//testtiming:keep`, and the marker is absent.

### [BLOCKING] A duplicate row in TestPrepareRunSettings

**Location:** internal/configreg/runsettings_test.go:77

**Issue:** The row at line 77 and the row at line 207 sit in the same table and pass the same `beam` settings with the same `wantDocs`, so one covers no behavior the other does not.

### [MEDIUM] The carry into seeded runs misses other machines and an early board save

**Location:** internal/dobbyengine/reinterpret.go:222

**Issue:** `carryLegacyBoardSettings` reads the retired keys through `boardengine.LegacyBeamSettings`, which reads only the raw keys on disk.
`Store.Load` folds those keys away, and the next save writes the fold.
No board verb runs `hubreconcile.Ensure`; only start verbs, clone, arm and adopt do.
An upsert or prune made by the new binary before the walk therefore leaves the carry nothing to read.
The walk applies the reinterpretation once per hub: `applyReinterpretations` skips it once `configengine.ReinterpretedMarker`, under the board's tracked `_lym/config`, exists.
The carry writes `configengine.LocalFile`, which sits under each worktree's untracked `.lym/config`, and it reads only the worktrees on the machine running the walk.
A run seeded before the upgrade and owned by another machine never gets its `merge-in` or its `land_notice` on.
A merge-parent task among them then lands through a squashed pull request, and the beam loses the parent's commit as an ancestor.
The assistant reviewer found the early-save case.

### [MEDIUM] A beam member's line still overrides its land_notice off

**Location:** internal/dobbyengine/landnotice.go:165

**Issue:** `rememberRuns` sends a member's line whenever the beam's pull request is recorded, whatever `note.Requested` reads.
The amended board entry says a value set lower down always overrides, and no decision was added since round 1 to license the exception.
The parent holds this finding.

### [MEDIUM] A project's notice request can be lost before the carry

**Location:** internal/boardengine/store.go:297

**Issue:** `foldLegacyBeamFields` deletes a project's `pull_request` and `notify_on_land` from `Unknown` and returns, folding nothing, since a project has no run settings.
The request then survives only in board.json's raw key until the next save.
`LegacyBeamSettings` carries it only when the reconcile walk reads the board before that save, and nothing recovers it after.
