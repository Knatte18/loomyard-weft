---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    location: _lyx/plan/20-plan-write-archives-webster.md:20
    summary: Card 20 makes Plan-Write's rotation newly refuse (webster run busy, archive collision) but plans no refusal-spec.md rows for either, breaking PATTERN-refusal-way-forward
  - id: F2
    severity: BLOCKING
    class: consistency
    location: _lyx/plan/07-begin-batch-one-remedy.md:20
    summary: Card 7 plans a new table test over the five recorded states although TestBeginBatch_Refusals already is the table for begin-batch's existing-report refusal, breaking PATTERN-test-economy
  - id: F3
    severity: BLOCKING
    class: consistency
    location: _lyx/plan/11-reset-targets.md:37
    summary: Card 11 puts HEAD-moving assertions in websterengine's reset_integration_test.go although PlanReset is read-only, and plans the --batch pairing refusals at both the engine and the CLI, breaking PATTERN-test-economy
  - id: F4
    severity: MEDIUM
    class: scope
    location: _lyx/plan/14-retire-fresh-after-reset.md:52
    summary: Card 14 names the wrong callers of fingerprintMismatchWayForward (it has three call sites in PlanEditError, batchCardEditError and Run) and its ImpactSummary understates the ripple through exported PlanEditError
  - id: F5
    severity: MEDIUM
    class: scope
    location: _lyx/plan/02-batcher-orientation.md:60
    summary: Card 2 claims tools/tokencount callers stay compiling but StartFit.write reads Weights.MasterBase and is listed in no card, so the build breaks from card 2 until card 9
  - id: F6
    severity: MEDIUM
    class: decision
    location: _lyx/plan/12-reset-start-archives.md:35
    summary: Card 12 leaves open whether PlanReset's foreign-dirty-path refusal still runs in standalone mode, which decides whether the standalone tree outcome and the per-path keep-refusal way forward it states are reachable
  - id: F7
    severity: MEDIUM
    class: design
    location: _lyx/plan/11-reset-targets.md:38
    summary: Cards add refusal-spec rows but miss existing rows their change makes stale or overlapping (reset unknown --to and run busy, the batcher.yaml did not load row, the unfinished-batch rebaseline row mis-described as part of the card-set row)
  - id: F8
    severity: LOW
    class: consistency
    location: _lyx/plan/04-reconcile-migration-hook.md:23
    summary: Card 4's TestReconcileAll rows re-assert own-orientation-kept and other-entries-intact, behavior card 3's MigrateConfig table already pins
  - id: F9
    severity: LOW
    class: design
    location: _lyx/plan/08-computed-merriam-base.md:34
    summary: Card 8 has the CLI's executionBatches compute and fail on the Merriam base for every bracket verb although a recorded partition never uses it, contradicting its own enters-only-at-partition-time rule
  - id: F10
    severity: LOW
    class: design
    location: _lyx/plan/10-gitrepo-reset-keep.md:18
    summary: Card 10 creates the unexported webster resetKeep wrapper although its only consumer is card 12's standalone reset, so it sits unused and untested for two cards
  - id: F11
    severity: LOW
    class: scope
    location: _lyx/plan/04-reconcile-migration-hook.md:20
    summary: Card 4 changes lyx config reconcile behavior (migration, migrated envelope key) but lists neither configcli/doc.go nor the reconcile command's long help
  - id: F12
    severity: LOW
    class: scope
    location: _lyx/plan/06-webster-config-invalid.md:30
    summary: Card 6 generalizes the pre-run refusal to webster.yaml but only its batcher.yaml cases reach it through a test
  - id: F13
    severity: NIT
    class: scope
    location: _lyx/plan/05-loom-preflight-batcher.md:26
    summary: Card 5 adds AnchorPath validation to loomPreflightEntry but omits the validatedFields row in shedrecipe's entries_simple_test.go table
  - id: F14
    severity: NIT
    class: consistency
    location: _lyx/plan/12-reset-start-archives.md:32
    summary: Card 12 says the archive-only path runs no git yet lists the worktree's uncommitted paths, which reads git status
---

### [BLOCKING] Card 20 adds Plan-Write rotation refusals with no refusal-spec rows

**Location:** `_lyx/plan/20-plan-write-archives-webster.md:20`, its `**Edit:**` list.

**Issue:** The card has the stale-plan rotation call `ArchiveRunRecord` before it moves the plan, and says it "refuses while a run holds the run lock, which aborts the rotation before the plan moves".
That is a new way for a loom row to refuse, and `ArchiveRunRecord` has a second refusal (`exists at both ... and ...`, `internal/websterengine/archive.go`) that the rotation now also reaches.
`PATTERN-refusal-way-forward` binds the loom section of `contracts/specs/refusal-spec.md` and requires a new refusal to land with its row in the same commit; the decision record's first Constraint repeats it.
The spec already carries the PR-Rework twins of these two (`rework archive collision` and `rework archive: webster busy`, `refusal-spec.md:236-237`), which shows the convention, but the card's targets do not include `contracts/specs/refusal-spec.md`.
The card's test plan covers the seam error leaving the plan in place, but no row names the refusal or its way forward.

**Fix:** Add `contracts/specs/refusal-spec.md` to card 20's `**Edit:**` list and state in the `**Intent:**` the two loom rows it gains (Plan-Write rotation: webster busy, transient, wait for the run to finish then re-step; Plan-Write rotation: archive collision, correctness halt, remove the stale copy then re-step).

### [BLOCKING] Card 7 plans a second table for a refusal an existing table already covers

**Location:** `_lyx/plan/07-begin-batch-one-remedy.md:20`.

**Issue:** The card plans "one table test over the five recorded states" in `beginbatch_test.go`.
`TestBeginBatch_Refusals` in that file is already the table-driven test of begin-batch's entry refusals, and its header says it covers "a report already on disk for a terminal record ... or for a begun non-terminal record".
Two of the five states (terminal and begun non-terminal) have rows there today, and the card's own update of those rows is the right place for the other three.
A new test function for the same refusal is redundant under `PATTERN-test-economy`.

**Fix:** Turn the card's test plan into rows of `TestBeginBatch_Refusals`: reword the two existing rows and add rows for terminal `stuck`/`failed`, terminal `dead`, terminal `done` and the recovery-kind begun record, each asserting its exact remedy and that no other verb is named.

### [BLOCKING] Card 11 assigns the HEAD move to the wrong test and tests the pairing twice

**Location:** `_lyx/plan/11-reset-targets.md:37`.

**Issue:** Two parts of the card's test plan break `PATTERN-test-economy`.
First, `reset_integration_test.go` is said to "move a real HEAD for each target", but `PlanReset` is read-only (the file header of `reset.go`: "Webster runs no mutating git") and `TestPlanReset` only asserts `plan.SHA`; the HEAD move belongs to `resetCmd` through fabric's `ResetPairCode`, which `TestResetCmd` in `internal/webstercli/reset_test.go` already drives per target with real HEAD, files and mutation-record assertions.
The card gives that file only "the flag pairing and the envelope's `target`", so the per-target move, the remote-branch update and `report-head` under a held run lock are left unasserted at the surface that performs them, while the engine file is asked to assert something it cannot do.
Second, the `--batch` pairing refusals are listed in the engine table ("including the `--batch` pairing") and again in the CLI test ("covers the flag pairing"), although the pairing is validated once, in `PlanReset`.

**Fix:** Add the three targets as steps of `TestPlanReset` (resolution and every new refusal asserted on the returned plan or error) and as steps of `TestResetCmd` (HEAD, files and `mutations` for each target, and `report-head` accepted under a held lock while `start` is refused), keep the held-lock acceptance and the pairing refusals in one place, and limit the CLI test to what only the verb adds (the `--batch` flag reaching `PlanReset`, the `target` and `sha` envelope keys).

### [MEDIUM] Card 14 mis-states who calls fingerprintMismatchWayForward

**Location:** `_lyx/plan/14-retire-fresh-after-reset.md:52` and the `**ImpactSummary:**` at line 62.

**Issue:** The card says "`fingerprintMismatchWayForward` takes the re-entry step, `BeginBatch` passes `lyx webster run` and `Run` passes `deps.reentryStep()`", and the ImpactSummary says it "gains a re-entry parameter its two callers pass".
The function has three call sites, none in `BeginBatch`: `PlanEditError` (`fingerprint.go:116`), `batchCardEditError` (`fingerprint.go:141`) and `Run` (`runlevel.go:463`).
`PlanEditError` is exported and called by `BeginBatch`, `RecordBatch`, `PersistRecoveryTerminal` and `webstercli`'s validate, so a new parameter ripples to callers the card lists no symbol for (`webstercli/validate.go` is in no card).
The ImpactSummary therefore states a false blast radius, and a writer following the Intent literally cannot thread the parameter through `BeginBatch`.

**Fix:** List `PlanEditError` and `batchCardEditError` in the card's `**Edit:**` targets, state which of them takes the re-entry step and which hard-codes the plain `lyx webster run`, name its remaining callers (including `webstercli`'s validate) in the ImpactSummary, and correct the Intent's caller sentence.

### [MEDIUM] Card 2 leaves tokencount uncompilable until card 9

**Location:** `_lyx/plan/02-batcher-orientation.md:60`.

**Issue:** The card says callers "stay compiling" and names only `startEstimate` and `Calibration.addRun` in `tools/tokencount`.
`StartFit.write` (`tools/tokencount/calibrate.go:481`) formats `profile.MasterBase`, where `profile` is a `batcher.Weights`; after card 2 renames the field it no longer compiles.
Card 9 lists `StartFit`, `WriteMarkdown` and `StartFit.Unusable` but not the `write` method either.
Cards 3 to 8 run between card 2 and card 9, so the tree does not build at those batch boundaries, which the plan's "Callers stay compiling" sentence says it does.

**Fix:** Add `tools/tokencount#StartFit.write` to card 2's `**Edit:**` targets (read `Orientation`, print the label the test expects) and to card 9's, where its printed text is reworded.

### [MEDIUM] Card 12 does not say whether standalone keeps PlanReset's dirty-path refusal

**Location:** `_lyx/plan/12-reset-start-archives.md:35-36`.

**Issue:** D1 says every target keeps `PlanReset`'s guards including foreign dirty paths, and D4 says standalone keeps "every uncommitted change git can carry across the move" and surfaces keep's refusal with a per-path way forward for "a path it did not [write]".
Card 12 has standalone call the planned move after `PlanReset` and leaves `PlanReset` unchanged for non-archive-only plans, so its `dirtyTrackedPaths` refusal fires first and refuses on every tracked change outside the run's own writes, before `git reset --keep` is reached.
Under that reading the card's standalone tree outcome ("standalone keeps every change git carries across") is false for foreign tracked changes, and the "commit or restore for any other" branch of the keep-refusal way forward is unreachable.
Which behavior is meant changes what the card lets through in standalone, which the rubric's attack-surface question requires the Intent to state.

**Fix:** State in the card whether `PlanReset` skips the foreign-dirty refusal in standalone (the keep form being the guard, as D4's Bound says) or applies it, and make the per-mode tree sentence, the keep-refusal way forward and the matching `reset_integration_test.go` and `refusal-spec.md` rows follow that choice.

### [MEDIUM] Refusal-spec row edits are described loosely, so stale or overlapping rows survive

**Location:** `_lyx/plan/11-reset-targets.md:38`, `_lyx/plan/06-webster-config-invalid.md:32`, `_lyx/plan/16-amended-cards-rebaseline.md:25`.

**Issue:** The same gap recurs in three cards, so it is filed once as a method finding.
Card 11 says the spec "gains a row per new refusal", but two existing rows become wrong: `reset: unknown --to` (`refusal-spec.md:113`) names only `start` and `pre-fix`, and `reset: run busy` (`refusal-spec.md:114`) has no `report-head` exemption.
Card 6 "gains the config-invalid and config-unreadable rows" while the webster section already holds `batcher.yaml did not load` (`refusal-spec.md:46`) with the same trigger and way forward, which leaves two rows for one refusal.
Card 16 says the spec "drops the unfinished-batch reason from the rebaseline card-set row", but the refusal that disappears has its own row, `rebaseline: begun card of an unfinished batch` (`refusal-spec.md:51`), which the card never deletes.
A reworded or removed refusal with a stale row breaks the "row lands in the same commit" half of `PATTERN-refusal-way-forward`.

**Fix:** In each card, name the existing row it rewrites or deletes: card 11 rewrites `reset: unknown --to` and `reset: run busy`, card 6 rewrites `batcher.yaml did not load` into the config-invalid row, card 16 deletes the `rebaseline: begun card of an unfinished batch` row and trims the matching clause of the card-set row.

### [LOW] Cards 3 and 4 pin the same migration behavior twice

**Location:** `_lyx/plan/04-reconcile-migration-hook.md:23`.

**Issue:** Card 3's table test over documents already asserts a profile carrying both `master_base` and `orientation` keeps its own `orientation`, and that an operator profile's other entries are carried whole and the output loads through `Active`.
Card 4's extension of `TestReconcileAll` asserts the same two properties through `ReconcileAll`.
What only the wiring adds is that the hook runs, the dry run writes nothing, apply writes the migrated bytes and `migrated` appears in the result; `PATTERN-test-economy` asks for one test per behavior.

**Fix:** Keep the `ReconcileAll` rows to the wiring (dry run reports and writes nothing, apply writes a file `batcher.Active` loads, an absent file is seeded and not migrated) and leave the own-orientation and carried-whole assertions to card 3's table.

### [LOW] Card 8 computes the Merriam base on paths that never use it

**Location:** `_lyx/plan/08-computed-merriam-base.md:34`.

**Issue:** The card says the base enters "only at partition time" and has `Run` compute it only where it forms a new partition, but it has the CLI's `executionBatches` compute it "from `c.geom`" with a failure refused as transient.
`executionBatches` is called by validate, begin-batch, record-batch, recover-batch and await-batch, and every call after first init has a recorded partition, where `ExecutionBatches` maps the recorded batches and never reads the base.
Each of those verbs would read `CLAUDE.md`, the Master template and the overview for nothing and could refuse on a read error on a path that cannot use the value.

**Fix:** Compute the base in `executionBatches` only when `st` is nil (validate's first-init partition), and pass the zero `StartBase` otherwise; state that in the Intent.

### [LOW] Card 10 creates a webster helper that only card 12 uses

**Location:** `_lyx/plan/10-gitrepo-reset-keep.md:18`.

**Issue:** `resetKeep` is an unexported wrapper in `gitwrap.go` whose doc comment says it is "used only by the standalone reset (card 12)".
The spec rule is that a helper serving one surface is bundled into that surface's card; here it sits in the gitrepo card, no card tests it for two cards, and card 12 lists it under `**Uses:**` as if it were another card's public symbol.

**Fix:** Move the `resetKeep` creation from card 10 to card 12 and drop it from card 12's `**Uses:**`.

### [LOW] Card 4 changes lyx config reconcile without touching its docs

**Location:** `_lyx/plan/04-reconcile-migration-hook.md:20`.

**Issue:** The card adds a migration step to `lyx config reconcile` and a `migrated` key to its JSON.
`internal/configcli/doc.go` describes `reconcile` ("drops or fills keys across every module against its template"), and the command's `Long` text in `configcli.go` describes what it compares and writes; neither is in the card's targets.
The repo's docs rule requires the module doc to land in the same commit as a change of observable CLI behavior.

**Fix:** Add `internal/configcli/doc.go` and the `reconcile` command's long help to the card's targets and state the migration and the `migrated` key there.

### [LOW] Card 6's webster.yaml refusal is not reached by a test

**Location:** `_lyx/plan/06-webster-config-invalid.md:30`.

**Issue:** Card 6 sends `websterengine.LoadConfig` failures through the same refusal builder and says a `webster.yaml` content failure carries `config_invalid: true` and an unreadable one does not.
The only pre-run test it plans, `TestPersistentPreRunE_BatcherSelection`, exercises `batcher.yaml`; `config_test.go` asserts the sentinel at the engine, not the CLI refusal.
`PATTERN-refusal-way-forward` asks that a test reach each new refusal, and the `webster.yaml` call site of the builder is a separate path.

**Fix:** Add a malformed `webster.yaml` step and an unreadable `webster.yaml` step to the same pre-run test, asserting `config_invalid` present and absent respectively and `state.json` unchanged.

### [NIT] Card 5 omits the entry table row for the new AnchorPath check

**Location:** `_lyx/plan/05-loom-preflight-batcher.md:26`.

**Issue:** `loomPreflightEntry` gains `requireAbsRoot("LoomPreflight", "AnchorPath", ...)`, and `simpleEntryCases` in `internal/shedrecipe/entries_simple_test.go` lists each entry's validated fields (`LoomPreflight` has `StatusPath` and `StatusLockPath`).
The card lists `loomrecipe/shape_test.go` and the loomshed tests but not this table, which then understates what the entry validates and skips the blank-field row.

**Fix:** Add `internal/shedrecipe/entries_simple_test.go` to the card's targets and `AnchorPath` to the `LoomPreflight` row's `validatedFields`.

### [NIT] Card 12 says "runs no git" but reads git state

**Location:** `_lyx/plan/12-reset-start-archives.md:32`.

**Issue:** The archive-only path "runs no git", yet the same card has every reset to start list `DirtyPaths` after the move, and `DirtyPaths` runs `git status`.
The decision record has the same two statements, so the card inherits the contradiction instead of resolving it.

**Fix:** Say "runs no mutating git" and note that the uncommitted listing is a read.

## Focus directive verdicts

- Card 11 `Intent` (report-head under a held lock): it states the skip and all five D1 bounds (begun and non-terminal, report on disk and parsing, descends from `StartSHA`, no later batch with a record, no live recovery strand) and names the out-of-order in-session fork as the unchecked residue. Not a defect; its test plan is F3.
- Cards 12 and 14 (archive-only path): the card states what is dropped, that it runs no git, that the record is moved and never deleted, that lock, merge, foreign-branch and no-state refusals stay, and that recovery strands are removed first. Not a defect beyond F6 and F14.
- Cards 16, 17 and 19: each `Intent` states its bound (in-flight batch only, `--card` named only, at most one forced recovery per amendment, overview change confined to the Card Index). Not a defect.
- Card 6 and the Master ladder: the `Intent` states the rung fires only on the parse-or-validation case and never on an unreadable file. Not a defect; the test gap is F12.
- Fidelity: every Decision D1 to D13 and every Constraint is carried by some card, except the Plan-Write rotation's refusal rows (F1). The `reset-archive-is-the-fresh-archive` departure rests on a true premise, since `Run`'s fresh branch archives through `archiveStateFile`, `archiveReportsDir` and `clearRenderedPrompts` (`runlevel.go:470-483`) while `ArchiveRunRecord` moves the whole webster directory to a told destination. `orientation-stays-measured` matches the record's conditional (31400 stands when no measured runs are read). No card carries scope the record does not license.
- Card order: cards 11 to 14 use `ResetPairCode` (existing), the keep-reset (card 10) and the archive factoring (card 12) only after they are added; card 17 follows card 16 and card 19 follows card 18. The one ordering defect is the compile break in F5.
- Verify coverage: the module is a single Go module, so `./...` covers `tools/tokencount`, `internal/loomshed`, `cmd/lyx` and `contracts/stencils`; the section follows the template's order. Not a defect.
- Test economy: F2, F3 and F8; the remaining test plans extend existing tests.
- Granularity: cards 2, 3, 4, 8 and 9 do not split a private helper or an inseparable constructor, and each cross-package public symbol has its card; the one misplaced helper is F10.
- `ImpactSummary`: real conclusions except card 14 (F4).
- `Custom` and `Prosa`: no card uses either type.
