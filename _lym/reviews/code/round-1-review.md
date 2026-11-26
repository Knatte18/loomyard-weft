---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/gatecli/memoryguard.go:288
    summary: "Config keys are still printed and written bare or without their module, including most sites in the parent's added list, against D3, PATTERN-config-key-names and the entry that orders them named before landing"
    fix: "Sweep by method, not by this list: take every key of every module's template and search all Go messages, flag help, comments, package docs and docs/ for it, then name each in full, `<module>.<dotted path>`, rendering messages through configengine.KeyName from one named constant per key and updating the tests that assert the texts. CLI command forms such as `lym config gate --set k=v` keep the CLI form. Known sites under internal/: gatecli/memoryguard.go:220,288,299,304, gatecli/watchdog.go:156,168, gatecli/redundancy.go:49,54, gatecli/mutatecheck.go:29, gatecli/testmap.go:323,533, gatecli/tierrun.go:295-329, gatecli/scope.go:358, batcher/registry.go:30-45, batcher/config.go:129,133, batcher/estimate.go:155-192, battencli/overviewdobby.go:38,165, battencli/cli.go:513,520, battencli/arm.go:117, beamcli/land.go:276,290, beamcli/pr.go:326,422,459,486, webstercli/fixplan.go:186, webstercli/fixverify.go:221, webstercli/recoverbatch.go:401, hibernatecli/cli.go:112, preflightshed/preflight.go:99, weaverengine/config.go:90,96,100, sheddriver/tackler.go:201, shedarbiter/call.go:194, burlerengine/fix.go:396, loomshed/plancontext.go:76,141. Comments, help and docs: loomcli/validate.go:409, loomcli/doc.go:59-62, loomcli/landingdeps.go:459,498,642, loomcli/circling.go:249, loomcli/darnwiring.go:287, loomcli/hardenwiring.go:64, hardencli/cli.go:286, loomengine/review.go:57-126, driftengine/doc.go:16,90,96, hardenshed/doc.go:10, docs/overview.md:1693,1695. Check reportcli too. shellguard's literal is the one declared exception."
    origin: lens:code-scope
  - id: F2
    severity: MEDIUM
    class: consistency
    location: internal/configcli/configcli.go:718
    summary: "Help text and docs still say only the reconcile verb and --unset remove a key the template lacks, and that only a start verb runs the hub walk"
    fix: "In internal/configcli/configcli.go (reconcile help, about lines 716-721) and internal/configcli/doc.go:70, add that every reconcile moves a key a module's move table names, or drops it when null or when its new path is set. In docs/overview.md:360, docs/shared-libs/configengine.md:148 and internal/preflightshed/doc.go:14, say that every verb that loads the loom or tapestry config runs the walk, not only a start verb."
    origin: lens:code-affected-set
  - id: F3
    severity: MEDIUM
    class: design
    location: internal/hubreconcile/walk.go:137
    summary: "No test fails if the walk mails its move lines only on success, the behaviour the parent's added entry orders as defect (5)"
    fix: "In internal/hubreconcile/ensure_integration_test.go add a case, with its seam in export_test.go, where the hub base loom.yaml holds code_max_bounces, the walk fails after the hub-wide commit (for example in a worktree reconcile), and the orch mail still holds the moved-keys notice."
    origin: lens:code-tests
  - id: F4
    severity: MEDIUM
    class: design
    location: internal/battencli/arm.go:300
    summary: "The reconcile-before-load calls in batten's arm, fabric merge-in, beam dobby, webster wiring and shuttleparity have no test, so removing or reordering any of them fails nothing"
    fix: "Add a stale-stamp row per site to that package's existing hub test, asserting a hub base loom.yaml old key is nested after the verb: internal/battencli/arm.go:300, internal/fabriccli/merge_verbs.go:149 (assert the walk ran before MergeIn), internal/beamcli/cli.go:718, internal/webstercli/wiring.go:120, internal/shuttleparity/seats.go:180."
    origin: lens:code-tests
  - id: F5
    severity: LOW
    class: design
    location: internal/yamlengine/move.go:79
    summary: "MoveKeys appends moved keys in sorted old-name order, so a YAML alias can land before its anchor and the output no longer parses"
    fix: "Make MoveKeys keep every anchor ahead of its aliases, for example by expanding alias nodes into copies of their anchored value before moving, and add a TestMoveKeys row with `code_max_bounces: &n 2` followed by `code_hard_max_bounces: *n` that asserts the output parses to both values."
    origin: lens:code-correctness
  - id: F6
    severity: LOW
    class: design
    location: internal/loomengine/config_test.go:107
    summary: "The move-table test iterates the table under test, so a D1 old key missing from loomMovedKeys or tapestryMovedKeys fails no test"
    fix: "Assert each table against a literal old-to-new map written from D1 (or against the flat key list of the pre-move templates) in TestLoadConfig_Loads and TestLoadTapestryConfig."
    origin: lens:code-tests
  - id: F7
    severity: LOW
    class: design
    location: internal/configengine/set.go:65
    summary: "Neither side of the area-named old key branch of Set has a test row"
    fix: "Add two TestSet rows over areaMoves: `area={model: x}` is written, and `area=x` is refused naming `--set area.model=x`."
    origin: lens:code-tests
  - id: F8
    severity: LOW
    class: design
    location: internal/configreg/runsettings.go:171
    summary: "No test covers MoveSettings dropping a null old key or an old key whose new path is set, the D5 rule D6 and D7 share"
    fix: "Add TestMoveSettings rows for `{loom: {code_max_bounces: null}}` and `{loom: {code_max_bounces: 2, code: {max_bounces: 3}}}`, asserting the result and the drop line."
    origin: lens:code-tests
  - id: F9
    severity: LOW
    class: design
    location: internal/configsync/configsync.go:209
    summary: "The two new error attributions, to the prime's file for a seed that cannot move and to the template base whose blocks cannot move, have no test row"
    fix: "Add two TestReconcile_FailureNamesItsFile rows through ReconcileHubWideAt, one with a prime loom.yaml holding `code: scalar` beside `code_max_bounces`, one with a profiles template base holding such a block, each asserting the FileError's path."
    origin: lens:code-tests
  - id: F10
    severity: LOW
    class: design
    location: internal/configreg/configreg_test.go:391
    summary: "TestLookup is listed as contained by TestPrepareRunSettings but pins each module's OpenMaps list, which that test never asserts"
    fix: "Add `//testtiming:keep pins each module's registered OpenMaps and found flag, which TestPrepareRunSettings never asserts` directly above `func TestLookup`."
    origin: handler
  - id: F11
    severity: LOW
    class: design
    location: internal/configreg/configreg_test.go:308
    summary: "TestModules_LocalLayerRefusals is listed as contained by TestLayerRunSettings but pins per-module local-layer verdicts that test never asserts"
    fix: "Add `//testtiming:keep pins per-module local-layer verdicts and the refusal naming file and key, which TestLayerRunSettings never asserts` directly above `func TestModules_LocalLayerRefusals`."
    origin: handler
  - id: F12
    severity: NIT
    class: design
    location: internal/configengine/config_test.go:616
    summary: "TestKeyName is contained, since the full-name format it asserts is already asserted through the refusals in TestEdit and TestLoad_LayerStack"
    fix: "Delete TestKeyName from internal/configengine/config_test.go."
    origin: handler
  - id: F13
    severity: NIT
    class: design
    location: internal/configsync/configsync.go:531
    summary: "moveBlocks joins profile and recipe names with a dot that MoveKeys splits again, so a profile whose name holds a dot is silently left flat"
    fix: "Address the block by its two segments instead of a joined dotted string (let configreg.MoveSettings and yamlengine.MoveKeys take the prefix as segments), or report an error naming the profile; add a row with a profile named `v1.2`."
    origin: lens:code-correctness
---

The move mechanism (D4 to D9) reads as correct in the paths traced.
The blocking item is D3 and the parent's added bare-key entry: most of the listed packages are not in the diff at all.

No gate or test was run in this round; every claim rests on reading and searching.

On `lym config <module> --unset` with several old keys in one file:
removing all of them in one command works, as the parent's added entry states.
Removing only one can be refused and rolled back, at the default layer by the strict load of the file that remains.
That entry settles the one-command route, so no finding is raised.

Declared PATTERN changes landed.
The merge commit's `dobbyengine/tasksetting.go` conflict keeps both sides.

### [BLOCKING] Config keys are still named bare or without their module

**Location:** `internal/gatecli/memoryguard.go:288` and the sites the fix lists.
**Issue:** The parent's added entry says the run names every listed bare key in full before it lands.
`gatecli`, `hibernatecli`, `shedarbiter` and others still print names such as `memory_quit_mb`, `resume_on_wake` and `arbiter_max_spawns`.
Comments and package docs, some added by this diff, name keys without their module, against `PATTERN-config-key-names`.
The cause is that no sweep compared message and prose strings with the template keys.

### [MEDIUM] Help and docs describe the old reconcile behaviour

**Location:** `internal/configcli/configcli.go:718`
**Issue:** The reconcile help, which D4 names as a way forward, does not mention the move and says other reconciles keep such keys.
Docs still say a start verb runs the walk.

### [MEDIUM] Mailing after a failed walk is untested

**Location:** `internal/hubreconcile/walk.go:137`
**Issue:** The only assertion on the moved-keys notice is over a successful walk.

### [MEDIUM] Five reconcile-before-load sites are untested

**Location:** `internal/battencli/arm.go:300`
**Issue:** D5 requires each call, and only the loom, shed and beam land entries have a stale-stamp test.

### [LOW] Moving can break alias order

**Location:** `internal/yamlengine/move.go:79`
**Issue:** `code_hard_max_bounces` moves before `code_max_bounces`, so an alias on the former precedes its anchor.
The reconcile that the load refusal names then fails on that file.
This was traced by reading, not run.

### [LOW] Move tables are checked only against themselves

**Location:** `internal/loomengine/config_test.go:107`
**Issue:** A missing entry passes, and that old value would be neither moved nor refused.
A comparison of the base templates' keys with the tables found no missing entry today.

### [LOW] Area-named branch of Set is untested

**Location:** `internal/configengine/set.go:65`
**Issue:** Every moved-key row of `TestSet` uses `old_key`.

### [LOW] Drop outcomes of MoveSettings are untested

**Location:** `internal/configreg/runsettings.go:171`
**Issue:** Only the moved line is asserted.

### [LOW] Move error attributions are untested

**Location:** `internal/configsync/configsync.go:209`
**Issue:** The one new failure row drives `ReconcileAll` only.

### [LOW] TestLookup needs a keep mark

**Location:** `internal/configreg/configreg_test.go:391`
**Issue:** The redundancy report lists it as contained, but coverage cannot see its `OpenMaps` assertions.

### [LOW] TestModules_LocalLayerRefusals needs a keep mark

**Location:** `internal/configreg/configreg_test.go:308`
**Issue:** The redundancy report lists it as contained, but coverage cannot see its per-key verdicts.

### [NIT] TestKeyName is redundant

**Location:** `internal/configengine/config_test.go:616`
**Issue:** Refusal assertions elsewhere already pin the rendered full name.

### [NIT] Dotted profile names are not moved

**Location:** `internal/configsync/configsync.go:531`
**Issue:** No name rule seen forbids a dot, and such a block is skipped with no line.

## Rejected

- code-correctness F1 (a prime-seeded `profiles.yaml` is never moved): unreachable, since the `profiles` registry row is hub-wide with no `OpenMaps` and no `SeedsFromPrime`, so `ReconcileHubWideAt` never reads a prime copy for it.
- code-tests aside (one unmovable board entry fails the carry for every entry): the carry is one board write by D7, which asks that a failure leave the board as it was and retry, and the logged error names the entry.
