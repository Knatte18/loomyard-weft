---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/hardenshed/archive.go:40
    summary: "The round archive is written under the worktree root but committed with anchor-relative paths, so on a hub whose anchor is not `.` the archive lands outside the anchor's `_lym` and the commit fails or commits nothing."
    fix: "In `internal/hardenshed`, rename `ArchiveDeps.WorktreeRoot` to `AnchorRoot` and join `_lym/harden/...` under it in `ArchiveRound` and `ArchiveFinalLedger`. Add `AnchorPath string` to `Campaign`, set it from `location.AnchorPath()` in `internal/loomcli/hardenwiring.go#hardenCampaign`, and have `Campaign.archiveDeps` pass it as `AnchorRoot`. Keep `Campaign.WorktreeRoot` for the package and sandbox checks. Make `TestArchiveRound` and `TestWrapRow` use an anchor root that differs from the worktree root."
    origin: lens:code-correctness
  - id: F2
    severity: MEDIUM
    class: design
    location: internal/shedrecipe/entries_bouncer.go:361
    summary: "Nothing pins that a Bouncer built with `harden_round` fills `LiveRed` and `OnHalt` from the campaign, or that the recipe's `auto_accept_setting` and `fixer_fan_setting` values resolve to `hardener.yaml`; deleting those lines passes every test and drops the red-live auto-accept bound and the halt reap."
    fix: "Add rows to `TestHardenRoundKey` in `internal/shedrecipe/entries_harden_test.go`. One builds a Bouncer row with `harden_round` over a campaign whose live report is red, drives a spent-budget auto-accept and expects an escalation with cause budget. Another drives a circling or budget halt and expects the campaign's `Reap` to run at `ReapHalt`. Add a `configreg` or `loomcli` test that resolves `hardener.auto_accept` and `hardener.reviewer_fan` from a hub `hardener.yaml` and not from `loom.yaml`."
    origin: lens:code-tests
  - id: F3
    severity: MEDIUM
    class: design
    location: internal/hardenshed/round_test.go#TestWrapRow
    summary: "The fix-row test only checks that `harden-strays-1.json` exists in the archive, so moving the archive before the end-of-row reap, or archiving a stray report without the end entry, passes; D7 and its acceptance criterion require the archived report to hold the end-of-row reap's stray."
    fix: "In `FixRowWritesTheCommitsReapsAndArchives`, read `harden-strays-1.json` from the archived round directory and assert it holds the entry of the `reap-end` call, with its row and `ReapEnd` point, beside the start entry."
    origin: lens:code-tests
  - id: F4
    severity: LOW
    class: design
    location: internal/hardenshed/live.go:646
    summary: "A red live mark caused by a failed run with no `FAIL` or `panic:` line, by a run that started no test binary, or by a timeout leaves `Deferred` empty, so the facts print `Deferred by the red live report: none` beside a red mark and the carry-over sees nothing deferred."
    fix: "In `NewLiveGate`, when the mark is red and `report.Live.FailingLines` is empty, set `report.Deferred` to one line naming the `harden-live` run as failed or started no test binary and pointing at the report path. Add a `TestLiveGate` row for a passing run that starts no binary and expect that line in `wantDeferred`."
    origin: lens:code-correctness
  - id: F5
    severity: LOW
    class: design
    location: internal/hardenshed/campaign.go:286
    summary: "An empty `reviewer` or `fixer` list, settable from a task's `run_settings`, passes loading and Preflight; `Rotate` then yields no entries and the rounds run on an unspecified model with empty models records, so the convergence rule's model-change test cannot hold."
    fix: "In `CheckCampaign`, refuse a `Config.Reviewer` or `Config.Fixer` of length 0 through `in.refuseKey` with the `run_settings` write as the way forward, before the reviewer-repeat check. Leave `checkValues` alone, since a sparse local document legitimately omits the key. Add `ReviewerListEmpty` and `FixerListEmpty` rows to `TestCheckCampaign`."
    origin: lens:code-correctness
  - id: F6
    severity: LOW
    class: design
    location: internal/hardenshed/round.go:645
    summary: "A fix row resumed after a crash re-reads `Head` at resume, so commits the earlier call made fall outside `RoundCommits.Commits` and the facts under-report the round's fix commits."
    fix: "In `roundRow.Call`, for a fix row read the round's existing `harden-commits-<round>.json` or a new `harden-head-<round>.json` first and reuse its recorded head; write the head at first entry before `inner.Call`. Add a `TestWrapRow` row that resumes a fix row over a recorded head and expects `RoundCommits.Head` to keep it."
    origin: lens:code-correctness
  - id: F7
    severity: LOW
    class: design
    location: internal/gatecli/run.go:659
    summary: "`--copies` expands every run that passes through `runWithSlots`, vet-step and guard-step runs included, so a planned vet or guard step would run N times and lease N times its slots."
    fix: "In `runWithSlots`, set `copying = e.request.copies > 0 && !empty && !run.vetStep && !run.guardStep`. Add a `TestRunTierCopies` row with a guard-step run that expects one run and one started line."
    origin: lens:code-correctness
  - id: F8
    severity: LOW
    class: consistency
    location: docs/overview.md:1953
    summary: "Prose still lists the recipes Run-Shed can start without `hardener`, so a hardener task reads as impossible in the overview and in the orch role's instructions, against D1's rule that every closed recipe list admits it."
    fix: "Add `hardener` to the recipe list at `docs/overview.md:1953` and to the sentence `That inner run is a tapestry, arras, motif, darn or sampler` at `contracts/orchrole/orch-role.md:44`."
    origin: lens:code-affected-set
  - id: F9
    severity: LOW
    class: scope
    location: internal/gatecli/cli.go#newCommand
    summary: "The `lym gate test` help rewrites lines beyond the `--copies` addition: it drops `through the platform shell` and the sentence `Inside a slot an enclosing gate run holds it acquires no other`, which still describes behaviour, and rewords `--upto`; no decision asks for it."
    fix: "Restore the original `--upto` line, the `Each tier takes the gate slots it asks for, and runs its command through the platform shell, bounded by its timeout_min` line and the `Inside a slot an enclosing gate run holds it acquires no other.` line in the `lym gate test` long text, and keep only the added `--copies` paragraph line."
    origin: lens:code-scope
  - id: F10
    severity: LOW
    class: design
    location: internal/loomrecipe/hardenerrecipe_test.go#TestHardenerRecipe
    summary: "Five tests the redundancy report lists as contained pin what their covering tests never assert and carry no `//testtiming:keep` mark: the recipe's on_done path, ring edges and profiles; the interrupt-policy table in both directions; the recipe structural check; the whole-key-set classification; and the module-to-reinterpretation table."
    fix: "Add a `//testtiming:keep <reason>` line directly above each `func Test…` line, the reason naming what it pins. `TestHardenerRecipe` in `internal/loomrecipe/hardenerrecipe_test.go`: the hardener recipe's on_done path, ring edges and review profile, which `TestArrasRun_GotoIntoRaddleWrite` never asserts. `TestInterruptPolicies_MatchAssembledRows` in `internal/loomrecipe/interruptpolicy_meta_test.go`: the policy table against the assembled rows in both directions, which `TestDriftRows_EntryGate` never asserts. `TestRecipe_StructuralCheckHasNoFindings` in `internal/loomrecipe/recipe_test.go`: the structural check of every embedded recipe, which `TestAmendmentTarget` never asserts. `TestModules_EveryTemplateKeyClassified` and `TestModules_RegisterTheirReinterpretation` in `internal/configreg/configreg_test.go`: the whole key-set classification and the module-to-reinterpretation table, which `TestPrepareRunSettings` never asserts."
    origin: handler
  - id: F11
    severity: LOW
    class: design
    location: internal/gatecli/copies_test.go#TestRunTierCopies
    summary: "Three `--copies` behaviours have no test: a copies call carries no green record, `testRequest.words` forwards `--copies` to a detached re-run, and the prebuilt `lym` is built once for all copies with `mergeCopies` summing the launches."
    fix: "In `TestRunTierCopies` assert `exe.green` is untouched after a copies call. Add a `copies` row to `TestTestRequestWords_MainPackages`. Add a copies row whose tier sets `PrebuiltLym` and one asserting the merged record's `Launches` and exit code over two copies."
    origin: lens:code-tests
  - id: F12
    severity: LOW
    class: design
    location: internal/loomcli/hardenwiring_test.go
    summary: "The campaign's Preflight key read (each tier's `covers` added to the tested tiers, an absent gate config) and the segment settings beyond reviewer rotation (fixer rotation, the `judge` and `seed` overrides, an unresolvable model-spec naming its `hardener.yaml` key) have no test."
    fix: "Add a `loomcli` test of `hardenLiveKey` over a gate config whose tier lists `covers` and over an absent config. Extend `TestWire_HardenerRecipe` to check `Models.Pick` for the fixer list in rounds 1 to 9, and add rows for a `judge` and `seed` override and for a bad model-spec expecting the key name and way forward."
    origin: lens:code-tests
  - id: F13
    severity: LOW
    class: design
    location: internal/pairteardown/teardown.go:506
    summary: "The real sandbox-clear spawn (the root-exists check, the `selfexec.LymBinary` spawn, the wait and the output in the error) and the `/proc` kill of a real orphan have no test; the teardown test replaces the spawn with a fake and `TestSandboxVerbs` clears a root with no process."
    fix: "Add a `pairteardown` test of `clearWorktreeSandbox` over a stub `lym` and a worktree holding a sandbox root, covering success and a failing exit. Add one tier-1 test in `internal/hardencli` that starts a child with its cwd under the root and expects `clear` to end it and report it."
    origin: lens:code-tests
  - id: F14
    severity: NIT
    class: scope
    location: internal/boardcli/cli.go
    summary: "Two help-text rewordings trace to no decision: `notify_on_land` loses `at once if it has`, and the `lym config` long text drops `directly`."
    fix: "Restore `tell the orch when the run lands, at once if it has` in `internal/boardcli/cli.go` and `name it edits that module directly` in `internal/configcli/configcli.go#buildConfigLong`."
    origin: lens:code-scope
  - id: F15
    severity: NIT
    class: consistency
    location: pattern/PATTERN-gate-slot.md:96
    summary: "The Verify sites sentence lists the Code-Burler-Fix, Darn-Write and Drift-Verify gates without Harden-Burler-Fix, which the diff names two sections later."
    fix: "In `pattern/PATTERN-gate-slot.md` line 96, add the Harden-Burler-Fix `verify` and `live` gates to the sentence."
    origin: lens:code-affected-set
---

### [MEDIUM] The round archive is written outside the anchor on a non-root-anchor hub

**Location:** internal/hardenshed/archive.go:40
**Issue:** `ArchiveRound` and `ArchiveFinalLedger` join `_lym/harden` under `Campaign.WorktreeRoot`, which `hardenCampaign` sets to `location.WorktreePath()`. The commit goes through `fabricengine.CommitAnchoredPaths`, whose paths are anchor-relative. With an anchor other than `.` the files sit outside the anchor's `_lym` and the pathspec names nothing, so D7's archive is lost. Every test uses anchor `.`.

### [MEDIUM] The Bouncer's red-live bound and halt reap are not pinned by any test

**Location:** internal/shedrecipe/entries_bouncer.go:361
**Issue:** `TestHardenRoundKey` checks only that the campaign's facts reach the facts file. Removing the `LiveRed` or `OnHalt` assignment leaves every test green, although D6 makes the red-live bound and D5 the halt reap acceptance criteria. No test shows that the recipe's `hardener.*` setting keys resolve to `hardener.yaml`.

### [MEDIUM] The archived stray report is not checked for the end-of-row reap

**Location:** internal/hardenshed/round_test.go#TestWrapRow
**Issue:** D7 orders gates, then the end-of-row reap, then the archive, and its acceptance criterion says a stray appears in the archived report with its row and reap point. The test checks only that the file exists, so reordering passes.

### [LOW] A red live mark can carry no deferred finding

**Location:** internal/hardenshed/live.go:646
**Issue:** `Deferred` is the failing lines of the run. A red mark from a run with no `FAIL` line, no started binary or a timeout leaves it empty, so the facts say `Deferred by the red live report: none` beside a red mark. D4 says a spent-attempts red gate records its failure as deferred.

### [LOW] An empty reviewer or fixer list is accepted

**Location:** internal/hardenshed/campaign.go:286
**Issue:** `reviewerRepeat` returns no refusal for an empty list and `checkValues` does not refuse it. The rounds then run with the zero model choice and empty models records, which the convergence rule's reviewer-change test cannot judge.

### [LOW] A resumed fix row records the wrong starting head

**Location:** internal/hardenshed/round.go:645
**Issue:** `Head` is read at each entry of the fix row. After a crash mid-row the resumed call starts from the later head, so commits the first call made are missing from `RoundCommits`.

### [LOW] `--copies` also expands vet and guard step runs

**Location:** internal/gatecli/run.go:659
**Issue:** `copying` ignores the run's kind. A vet-step or guard-step run in a copies call would run N times under a lease of N times its slots.

### [LOW] Prose recipe lists still omit hardener

**Location:** docs/overview.md:1953
**Issue:** The overview's Run-Shed sentence and the orch role's inner-run sentence list the startable recipes without `hardener`, against D1.

### [LOW] The gate help is reworded beyond the `--copies` line

**Location:** internal/gatecli/cli.go#newCommand
**Issue:** The diff drops `through the platform shell` and the inherited-slot sentence and rewords `--upto`. None of it is asked for, and the dropped sentence describes behaviour that still holds.

### [LOW] Contained tests carry no keep mark

**Location:** internal/loomrecipe/hardenerrecipe_test.go#TestHardenerRecipe
**Issue:** The redundancy report lists five changed tests as contained. Checked against their covering tests, each pins an assertion the covering test lacks, so each needs a `//testtiming:keep` mark naming it: the hardener recipe shape against `TestArrasRun_GotoIntoRaddleWrite`, the interrupt-policy table both ways against `TestDriftRows_EntryGate`, the structural check against `TestAmendmentTarget`, and the two `configreg` tests against `TestPrepareRunSettings`.

### [LOW] `--copies` side behaviours are untested

**Location:** internal/gatecli/copies_test.go#TestRunTierCopies
**Issue:** No test covers the skipped green record, the forwarding of `--copies` in `testRequest.words`, the single prebuild shared by the copies, or the merged launch totals.

### [LOW] Preflight's gate-key read and the segment settings are only partly tested

**Location:** internal/loomcli/hardenwiring_test.go
**Issue:** `hardenLiveKey` adds each tier's `covers` to the tested tiers and handles an absent gate config, with no test. `TestWire_HardenerRecipe` checks reviewer rotation only, not the fixer list, the `judge` and `seed` overrides or the refusal of an unresolvable model-spec.

### [LOW] The real teardown spawn and the `/proc` kill are never run

**Location:** internal/pairteardown/teardown.go:506
**Issue:** The teardown test substitutes `clearSandbox`, and `TestSandboxVerbs` clears an empty root. The spawn through `selfexec.LymBinary` and the kill of a real orphan are untested.

### [NIT] Two unrequested help-text rewordings

**Location:** internal/boardcli/cli.go
**Issue:** `notify_on_land` loses `at once if it has` and the `lym config` long text drops `directly`. Neither follows from a decision.

### [NIT] A PATTERN-gate-slot sentence omits Harden-Burler-Fix

**Location:** pattern/PATTERN-gate-slot.md:96
**Issue:** The Verify sites sentence lists the other fix-row gates but not Harden-Burler-Fix, which the diff adds to the same file.

## Focus verdicts

- Reaper and sandbox: `Reap` ends recorded processes through `runproc.End`, and kills through `EndProcess` only a root-naming process no record names, sparing the caller and its ancestors. `StopServer` is called only for hubs in the recorded list, gone directories included. Nothing acts outside the root. Not a defect; F13 is a test gap only.
- `--copies`: one lease of N times the tier's slots replaces the tier run's own acquire, a lease above the pool is clamped and the clamp line is printed, and all refusals fire. A call inside an inherited slot is refused, and the live call drops the slot variable, so no nested deadlock. Findings F7 and F11 are small.
- Live gate: only `harden-live` sets the mark, a run that started no binary reads red, copies never fail the gate, `llm_driving` skips them, and tier 3 is refused at Preflight and by `--copies`. `PassOnCap` ends the row at spent attempts. F4 is the one gap.
- Auto-accept bound: both accept paths in the Bouncer check `LiveRed` and escalate with cause budget. The CONVERGED rule's green-gate and model-change tests live in the rubric text, as D6 says. F2 is a test gap.
- Model rotation: the lists expand cyclically to `hard_max_bounces`, rounds 1 to 9 take entries 1, 2, 3, 1, 2, 3, 1, 2, 3, and `reviewBudget` reads `max_bounces` from `hardener.yaml`. An empty list is the gap in F5.
- Config classification: the four Guard keys carry `HubOnlyWayForward`, and every other key is `CategoryNone`; the arm and verify scope read `hardener.yaml` and the test proves a `darn.yaml` edit changes nothing. Not a defect.
- Closed recipe lists: every list the record names admits `hardener`, and a grep of production code finds no other closed list; only prose lists remain (F8).
- Teardown: the clear spawns through `selfexec.LymBinary`, runs after the archive and before the removal, and keeps the worktree with its way forward; Finalize and the halts reap. Not a defect; F13 is a test gap.
- Loom contract: all four refusals and the unchanged check for an unsegmented build row are present and tested. Not a defect.
- Treadle, Tenter and crucible: no live reference remains; the leftover `crucible` mentions are the history comments, test names and records card 20 keeps. The `agents` reinterpretation strips both roles from `skills` and `efforts`, and the result loads.
- Round archive and facts: the order is gates, end-of-row reap, archive, and the facts carry models, live outcome, strays, deferred findings and fix commits. F1, F3 and F6 are the defects.
- Tests: no assertion was widened and no `Loosened-Check` is needed; the contained-test candidates are F10.
- Scope and PATTERN: the `PATTERN-child-processes` change lands as declared in `PATTERN.md` and its background file, and no other pattern file changes beyond what the new packages force. No card writes the hub gate values. The change description at `_lym/landing/summary.md` is absent, so its three `lym config gate --set` commands could not be checked.

## Rejected

- The record's `selfreport: false` key (from a lens): `loom.yaml` has no such key, and the diff sets `friction: ""`, which turns Tier 2 self-reporting off. Satisfying the record's wording would need a key that does not exist, so this is a point about the answer key, not the diff.
- Code-scope's note on the `docs/overview.md` tree line: licensed by D12 and card 20.
- Residual `crucible` comments in `internal/planglyph`, `internal/planparser` and `internal/shuttleengine/attach.go`: card 20 keeps them as history.
- The live gate's `lym gate test` spawn is not registered in `internal/runproc`: the starter waits on it, as verify gates that run the same call through a shell already do, so no new rule is broken.
- The recipe's `reviewer_fan_setting` and `auto_accept_setting` resolution gap raised separately by a lens: folded into the second finding.
