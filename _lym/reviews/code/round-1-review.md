---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: internal/shuttleengine/claudeengine/manifest.go:39
    origin: handler
    summary: "The diff declares a new constant localInstructionsFileName holding the literal CLAUDE.local.md, which heddleengine's claude-file target row and claudeengine/capabilities.go already spell, so the name is no longer declared once (PATTERN-code-conventions, Named constants)."
    fix: "Export one constant for the claude local instruction file name from internal/heddleengine/target.go, and use it in the claudeFileTarget row (File field, target.go:68). Delete localInstructionsFileName in claudeengine/manifest.go and use the heddleengine constant at manifest.go:188 and :190 and in claudeengine/manifest_test.go (the writeUserSetupFile call that names it). Also use it in the literal at claudeengine/capabilities.go:35. Point codexengine's agentsOverrideFile (instructionfiles.go:21), which the new codexengine/manifest.go reads at line 153, at an equivalent exported heddleengine constant for AGENTS.override.md."
  - id: F2
    severity: BLOCKING
    class: consistency
    location: internal/heddleengine/target.go:68
    origin: handler
    summary: "The diff adds two new hard-coded engine literals, Engine claude and Engine codex, on the claude-file and codex-file target rows; PATTERN-code-conventions says a change never adds another engine literal."
    fix: "In internal/heddleengine/target.go declare one named constant per engine name the table uses (claude and codex), and use them in the two new Engine fields on lines 68 and 69 and in the existing prompt rows that spell the same literals. Keep the engine value of a row a table field keyed by name, as the rule requires."
  - id: F3
    severity: MEDIUM
    class: design
    location: internal/shuttleengine/codexengine/manifest.go:300
    origin: lens:code-correctness
    summary: "Codex skips an omitted plugin hook whose Plugin is empty (hooks file unreadable) while Claude lists it as an omitted plugin-context item keyed by the hook name, so the Codex record's verdict misses the omission and the comparison reports Codex as lacking the Claude item."
    fix: "In setupCandidates (codexengine/manifest.go:300-307) drop the hook.Plugin empty-string skip. For a hook with an empty Plugin, list an omitted plugin-context item keyed by hook.Name with Reason hook.Reason, the same key claudeengine/manifest.go:346-349 uses, so Codex's verdict records it. Add a row to codexengine.TestPrepare_WritesTheManifest for an OmittedHook with no Plugin that asserts the omitted item and its verdict entry."
  - id: F4
    severity: MEDIUM
    class: decision
    location: internal/shuttleparity/prepare.go:19
    origin: handler
    summary: "ReferenceEngine and ComparedEngine hard-code the engine names claude and codex in production code that shuttlecli also uses, against PATTERN-code-conventions (no caller names an engine); the plan's card 6 asks for exactly these constants, so the plan and the constraint conflict and the constraint wins."
    fix: "The author decides and records it: either derive the reference and compared engine names from the engine set or shuttle config instead of constants in internal/shuttleparity/prepare.go and the uses in shuttlecli/manifest.go, producers_test.go and seats_test.go, or keep the constants and add a Pattern-Departure trailer on a commit, naming PATTERN-code-conventions, with the reason that the parity comparison is between two named engines by definition. Do not leave the conflict unrecorded."
  - id: F5
    severity: MEDIUM
    class: design
    location: internal/shuttleparity/report.go:43
    origin: lens:code-tests
    summary: "Nothing asserts that the verb's temporary directory is removed on an error path: TestReport checks removal on success rows only, and the unknown-seat row never creates an engine run directory to stat."
    fix: "In internal/shuttleparity/report_test.go TestReport, assert removal on every row. Have the report function's temporary directory observable on error rows too, for example by a failing Prepare row that records its run directory's parent before returning an error and an unknown-seat row that asserts the lymtemp manifest-report-* directory is absent afterwards, then run the os.Stat removal check before the wantErr return."
  - id: F6
    severity: MEDIUM
    class: design
    location: internal/shuttlecli/manifest.go:436
    origin: lens:code-tests
    summary: "manifestFields has no assertion for stripping each manifest's Verdict, for listing omitted plugin contexts under unrun by engine, or for printing missing as channel key strings; the stub reports hold no omitted item, so the required pass with plugin contexts listed unrun is unpinned."
    fix: "In internal/shuttlecli/manifest_test.go TestManifestCmd add a report whose Codex manifest holds one omitted plugin-context item and a non-empty Verdict. Add a row asserting exit 0 and an output holding an unrun entry naming that item under the codex engine and no verdict entries, and assert the missing row prints the item as channel key."
  - id: F7
    severity: MEDIUM
    class: design
    location: internal/shuttlecli/manifest.go:462
    origin: lens:code-tests
    summary: "manifestReportOf and the PersistentPreRunE wiring that sets manifestReport have no test, so replacing the NewWithSeams call with one that runs real plugin hooks and the sandbox probe, or dropping the no-shortname refusal, passes every test."
    fix: "Add a test in internal/shuttlecli/manifest_test.go that calls manifestReportOf over a fixture hub with a recorded shortname and asserts a seat's report holds both engines' manifests without the plugin hook runner or sandbox probe being run, and a second row for a hub with no shortname that asserts the refusal names the way forward. Reuse an existing shuttlecli hub fixture if one exists."
  - id: F8
    severity: MEDIUM
    class: design
    location: cmd/lym/main.go:180
    origin: lens:code-tests
    summary: "The production passage reader that converts heddle Passages and FileEntries into a shuttleengine.Standing is untested, so dropping FileEntries or the conversion fails nothing and production heddle file entries would vanish from the manifests and the comparison."
    fix: "Move the closure body into a named function in a package under test, or into cmd/lym with a test, and assert that over a heddle store holding one prompt entry and one file entry it returns both as Passage values with slug, text and engine key. Wire main.go to that function."
  - id: F9
    severity: LOW
    class: consistency
    location: internal/shuttleengine/claudeengine/manifest.go:332
    origin: lens:code-correctness
    summary: "The log message and the doc comment on resolvedCandidates say that a failed user-setup resolution lets the comparison name what the manifest then lacks, but Claude is the reference manifest, so fewer Claude items name nothing; the failure is caught only because Codex's Prepare refuses on the same source error."
    fix: "In claudeengine/manifest.go reword the doc comment at lines 310-314 and the logger.Warn text at line 332 to say that the launch manifest lists no instruction file above the working directory, listed skill or plugin context, and that the comparison itself fails because the Codex launch refuses on the same source error. Do not claim the comparison names the missing items."
  - id: F10
    severity: LOW
    class: design
    location: internal/shuttleengine/claudeengine/manifest.go:345
    origin: lens:code-tests
    summary: "The Claude branches that list an omitted plugin hook (hooks file unreadable), a failed user-setup resolution and a working directory with no repository root have no test row."
    fix: "In claudeengine/manifest_test.go TestPrepare_WritesTheManifest add rows for a plugin whose hooks file is unreadable (asserting an omitted plugin-context item and a verdict entry), for a resolution that errors (asserting no native instruction, skill or plugin items and the launch still succeeds), and for an anchor with no .git above it (asserting the CLAUDE.md key is exactly CLAUDE.md)."
  - id: F11
    severity: LOW
    class: design
    location: internal/shuttleengine/claudeengine/claudeengine.go:539
    origin: lens:code-tests
    summary: "Both Prepare doc comments say a failed manifest write refuses the launch, but no test makes the write fail and asserts the Prepare error."
    fix: "Add one row each to claudeengine.TestPrepare_WritesTheManifest and codexengine.TestPrepare_WritesTheManifest that makes the run directory unwritable for manifest.json (for example by planting a directory named manifest.json there) and asserts Prepare returns an error naming the way forward."
  - id: F12
    severity: LOW
    class: consistency
    location: internal/heddleengine/render_test.go:183
    origin: handler
    summary: "The redundancy report lists TestShowAndPassagesReadTheShippedLayerFirst as contained, but it pins what its covering tests do not assert: Show and Passages output of a prompt target under a shipped entry with the override on and off, while TestRender asserts only the rendered CLAUDE.local.md."
    fix: "Add a //testtiming:keep line directly above func TestShowAndPassagesReadTheShippedLayerFirst in internal/heddleengine/render_test.go, with the reason that it pins the Show and Passages output of a prompt target under a shipped entry with the lym override on and off, which TestRender and TestOverrides do not assert."
  - id: F13
    severity: NIT
    class: consistency
    location: docs/overview.md:1982
    origin: lens:code-affected-set
    summary: "The verb list lym shuttle run|open|state|await omits the new manifest verb, while the shuttle module entry earlier in the same file was updated to include it."
    fix: "In docs/overview.md line 1982 change the verb list to lym shuttle run|open|manifest|state|await, as the module entry at line 640 reads."
---

### [BLOCKING] A second declaration of the claude local instruction file name

**Location:** internal/shuttleengine/claudeengine/manifest.go:39
**Issue:** `localInstructionsFileName` repeats the literal that `heddleengine`'s `claude-file` row and `claudeengine/capabilities.go:35` already hold. `PATTERN-code-conventions` requires a name that can go stale to be one named constant declared once, and calls an existing literal debt that a change never adds to. The codex side keeps the same debt in `agentsOverrideFile`, which the new `codexengine/manifest.go` reads.

### [BLOCKING] New engine literals on the heddle file targets

**Location:** internal/heddleengine/target.go:68
**Issue:** The diff sets `Engine: "claude"` and `Engine: "codex"` on the two file target rows, adding engine literals that `PATTERN-code-conventions` says a change never adds. The existing prompt rows spell the same literals, so one set of named constants removes the debt for all rows.

### [MEDIUM] Codex drops an omitted plugin hook with no plugin name

**Location:** internal/shuttleengine/codexengine/manifest.go:300
**Issue:** An unreadable hooks file gives an `OmittedHook` with an empty `Plugin`. Claude lists it as an omitted `plugin-context` item keyed by the hook name. Codex skips it, so its record's verdict does not count the omission (D1: an omitted item counts as missing in the verdict), and `MissingItems` reports the Claude item as lacking on Codex.

### [MEDIUM] Engine names hard-coded in production code, against the plan's own card

**Location:** internal/shuttleparity/prepare.go:19
**Issue:** `PATTERN-code-conventions` says no caller names an engine. Card 6 of the plan, as the plan-review closing entry says to implement it, declares `ReferenceEngine` and `ComparedEngine`. The constraint wins over the plan, so the conflict is raised here for the author to settle and record. It is class `decision` at MEDIUM because the parity comparison is between two named engines by design, which the author may record as a departure.

### [MEDIUM] The verb's temporary directory removal is tested on success only

**Location:** internal/shuttleparity/report.go:43
**Issue:** D10 and the acceptance criteria require the directory to be removed on every path including errors. `TestReport` checks removal after success rows and returns early on error rows, so a leak on an error path passes.

### [MEDIUM] The verb's output fields have no assertion for unrun contexts or the stripped verdict

**Location:** internal/shuttlecli/manifest.go:436
**Issue:** The acceptance criterion that the verb passes with plugin contexts listed unrun rests on `manifestFields`' `unrun` map and the nulled `Verdict`. No stub report holds an omitted item, so neither is asserted.

### [MEDIUM] manifestReportOf is untested

**Location:** internal/shuttlecli/manifest.go:462
**Issue:** The D10 bound that the verb runs no plugin hook or sandbox probe depends on `manifestReportOf` calling `NewWithSeams` with `UnrunHooks` and `ConfiguredSandbox`. `TestManifestCmd` builds `shuttleCLI` by hand with a stubbed report, so the wiring is unpinned, and so is the no-shortname refusal.

### [MEDIUM] The production passage reader adapter is untested

**Location:** cmd/lym/main.go:180
**Issue:** Every test installs its own reader. A change that drops `FileEntries` from the production reader fails no test, and production heddle file entries would silently leave the manifests and the comparison.

### [LOW] The Claude resolution-failure comment claims more than the code does

**Location:** internal/shuttleengine/claudeengine/manifest.go:332
**Issue:** A failed resolution lists no native item on the reference manifest, which asks nothing more of Codex. The test still fails only because Codex's `Prepare` refuses on the same source error. The comment says the comparison names what the manifest lacks, which is false for the reference engine.

### [LOW] Untested Claude manifest branches

**Location:** internal/shuttleengine/claudeengine/manifest.go:345
**Issue:** The omitted-hook branch, the failed-resolution path and the no-repository path of `workDirInstructionKeys` have no row. The fixture always has a repository and no unreadable hooks file.

### [LOW] A failed manifest write refusing the launch is untested

**Location:** internal/shuttleengine/claudeengine/claudeengine.go:539
**Issue:** Both `Prepare` doc comments promise the refusal and no test makes the write fail.

### [LOW] A contained test pins what its covering tests do not

**Location:** internal/heddleengine/render_test.go:183
**Issue:** The redundancy report lists `TestShowAndPassagesReadTheShippedLayerFirst` as contained. `TestRender`'s override rows assert the rendered `CLAUDE.local.md` only, and `TestOverrides` asserts no `Show` or `Passages` output, so this test pins a prompt target's `Show` and `Passages` output under a shipped entry with the override on and off. It is kept and marked, not folded.

### [NIT] The overview's verb list omits the new verb

**Location:** docs/overview.md:1982
**Issue:** The list reads `lym shuttle run|open|state|await`, while the shuttle module entry at line 640 names `manifest`.

## Rejected

- Lens code-affected-set, LOW scope: the run-root readers other than `findRunByStrand` and `sweepOrphans` must skip `manifests`. Each of them calls `loadRunState`, finds no `run.json` and continues (`enginesrun.go`, `stepstop.go`, `carryover.go`, `attach.go`, `sessionread.go`, `agentwait.go`, `activity.go`, `waitmark.go`), and D1 names only the sweep and the strand lookup. The only effect is a lock file inside `manifests/`.
- Lens code-correctness, F1 as MEDIUM design (a swallowed Claude resolution error hides items from the comparison): lowered and reclassified as the LOW comment finding F9, because Codex's `Prepare` refuses on the same source error, so the parity test and the verb still fail loudly.
- Lens code-tests, F1 and F2 as separate findings: merged into F10, since both name untested Claude branches in one file.
- Handler: the `path+".lock"` lock paths of the manifest record and its copy. `PATTERN-lym-files` governs tracked-file locks, and the same spelling is the established form in `rundir.go` and elsewhere in the module.
- Handler: the edit-directive item is absent from the run's own record and added by `shuttleparity.PromptItems`. The plan's shared decisions settle this and the diff follows them.
- Lens code-scope reported no finding, and the handler's trace agrees.

## Focus departures

- Directive bullet 2 says the split-out renderers for `judgeCall` and `runSeedSpawn` are in `internal/burlerengine/review.go` and `internal/landingshed/fixreview.go`. Those two files in the range hold the seat-table entry points `SeatTables` and `FixReviewSeatTable`. The renderers are `RenderSeedPrompt` and `RenderJudgePrompt` in `internal/shedadapters/bouncer.go`, which the range does not touch. I confirmed that `runSeedSpawn` renders through `RenderSeedPrompt`, and I did not trace `judgeCall`'s use of `RenderJudgePrompt` beyond its definition, so the "prompts unchanged" question is answered for the seed producer and not stated for the judge producer.
