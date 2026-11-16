---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    origin: handler
    location: _lym/plan/13-run-reentry-and-resume.md:33
    summary: The no-spawn re-entry path records a stopped batch done without running its card gates at HEAD, the step D7 requires
    fix: In card 13's reenter bullet for any other batch stop, after the post-batch checks pass on a clean tree, run each card's card gate and the batch's batch gate at HEAD, the same calls batchGateSpec renders minus the row gates, through RunDeps.Verify (verifytree.Verify in held gate slots with the geometry's pause hold). Record the batch done with no spawn only when every one passes; a failing gate gives the batch its last-rung attempt. Add a TestRun_Reentry row where the post-batch checks pass but a card gate fails at HEAD, so the batch gets the last-rung attempt and is not recorded done.
  - id: F2
    severity: BLOCKING
    class: design
    origin: handler
    location: _lym/plan/02-weaver-config.md:34
    summary: weaver.yaml's verify_gate picks the gate key Weaver's Done rests on but is classified CategoryNone, so a local override can hollow the plan-level verify (PATTERN-config-layers)
    fix: In card 2, classify verify_gate as configengine.CategoryGuard and keep ladder and fixer_timeout_min CategoryNone. Move internal/configreg#TestModules_EveryTemplateKeyClassified from Uses to Edit, and replace the sentence saying the test needs no new row with one saying it gains wantLocalDenied["weaver"] = [verify_gate]. In card 2's Intent, state the bound — the local layer cannot change verify_gate, and only the tracked worktree file or the template sets it.
  - id: F3
    severity: BLOCKING
    class: consistency
    origin: lens:repo-invariants
    location: _lym/plan/01-shared-plan-conflict-path.md:45
    summary: runplan.ConflictRecordRel would be a production function no main package reaches; its only caller, websterengine.ConflictRecordRel, is allowlisted as test-only, so the test-only-code guard fails (PATTERN-test-only-code)
    fix: In card 1, drop the Create bullet plan:internal/runplan#ConflictRecordRel and its Intent line. Have websterengine.ConflictRecordRel join lymdirs.DotLymDirName, lymdirs.ImplementerDirName and runplan.ConflictRecordName with path.Join itself, so the one allowlisted test oracle stays the only reader.
  - id: F4
    severity: MEDIUM
    class: design
    origin: handler
    location: _lym/plan/13-run-reentry-and-resume.md:20
    summary: Card 13 has Run call resumeStrands and reenter before anything else, but neither card 12 nor card 13 lists Run as changed, so the wiring falls outside every card's scope
    fix: In card 13, add an Edit group listing plan:internal/weaverengine#Run. Add an Intent bullet saying Run, after taking its run lock and removing the stale conflict record, calls resumeStrands and then, when StopRecord is set, reenter, before it begins any batch.
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:guard-reach
    location: _lym/plan/13-run-reentry-and-resume.md:30
    summary: No card clears StopRecord, so after a successful re-entry a later re-step (a plan-conflict Stuck or a crash) runs reenter again over a batch already recorded done or a verify already passed
    fix: In card 13, have reenter clear StopRecord and save the state once the stopped batch is recorded done, or once the verify re-run passes. In card 12, have each batch begin clear any StopRecord under the state-mutation lease. Add a TestRun_Reentry row in which a re-step after a successful re-entry, followed by a plan-conflict Stuck, runs no second re-entry.
  - id: F6
    severity: MEDIUM
    class: consistency
    origin: lens:guard-reach
    location: _lym/plan/12-run-loop.md:43
    summary: Card 12 records a fixer's outcome but never marks its gatefixer.Record terminal through fixerStore.Save, and gatefixer.Await never does so itself, so SpawnOrAttach returns the ended record as live and no next fixer or same-rung retry spawns
    fix: In card 12's verify bullet, say that when Await returns a terminal result Run sets the fixer's Record.Status (and DeadReason) from that result, saves it through fixerStore.Save, and then sets FixerState.Outcome. Add a TestRun row in which a first fixer ends dead and the re-run verify still fails, so a second fixer spawns.
  - id: F7
    severity: MEDIUM
    class: design
    origin: handler
    location: _lym/plan/21-decision-notice-weaver.md:36
    summary: RecordDelivery writes Delivered into Weaver's state file from the decision verb's process while Run holds its own in-memory State, and Run's next save (endAttempt, card 10) overwrites it, so D13's delivery record is lost and the resume re-sends delivered entries
    fix: In card 10, and in card 12 wherever Run saves the state, have each save load the state file under the state-mutation lease and keep each attempt's and fixer's Delivered list as loaded, merged with the headings Run itself recorded, before it writes. Add a TestRun row in which a Delivered heading written to the file during an attempt survives that attempt's end.
  - id: F8
    severity: MEDIUM
    class: design
    origin: lens:guard-reach
    location: _lym/plan/14-weaver-archive-and-reset.md:25
    summary: Reset step 2 commits any uncommitted change under _lym/weaver or the plan directory whenever no state file exists, also when no Weaver run ever began, so lym weaver reset can commit unrelated plan edits; the card states no bound for this escape
    fix: In card 14, have Reset write a reset-pending marker under .lym/weaver/ before it moves, restores or archives anything, and remove it after the commit lands. Have step 2 finish a pending commit only while that marker exists, and otherwise refuse, naming that no Weaver run has begun. State in the Intent that the branch commits only what a reset this marker records left behind. Add TestReset rows for an absent state with plan edits and no marker (refuses and commits nothing) and with the marker (commits).
  - id: F9
    severity: MEDIUM
    class: consistency
    origin: handler
    location: _lym/plan/21-decision-notice-weaver.md:32
    summary: decisionNoticeInline(strand) and heldBackReason(strand, halted) are pure functions of their arguments today, but the Intent makes their answer depend on ActiveOrSetting answering Weaver, a signature change the Edit glyphs carry no re-sign arrow for
    fix: In card 21, re-sign the two Edit bullets as internal/loomcli#decisionNoticeInline -> func decisionNoticeInline(strand string, weaverActive bool) bool and internal/loomcli#heldBackReason -> func heldBackReason(strand string, halted shedengine.State, weaverActive bool) string. Add internal/loomcli#addedNoticeLines under Edit, since it calls decisionNoticeInline. In the Intent, say notifyLiveRuns computes weaverActive once from ActiveOrSetting and passes it down.
  - id: F10
    severity: MEDIUM
    class: design
    origin: handler
    location: _lym/plan/09-batch-attempt-strand.md:44
    summary: Cards 9, 10 and 11 plan per-helper tests of unexported functions whose behaviours card 12's TestRun covers at the exported Run surface, against PATTERN-test-speed's test economy
    fix: Drop TestClassifyAttempt and TestBatchGateSpec (card 9), TestEndAttempt (card 10), and TestRunVerify and TestWriteSummary (card 11), with their Create bullets and test paragraphs. In card 12, add to TestRun the rows those tests alone held — an OutcomeTimeout, an API-error loss and an unparseable report; the gate spec order of a two-card batch with the row's drift gate, and the last batch's main packages; a head_sha mismatch, an abbreviated head_sha and a clean parent merge; a done-check finding and a removed later target; a verify fail-then-pass recorded flaky, and a timeout not rerun; and summary.md's two sections. TestRun already holds the FAILED, failing-gate, unclean-tree, machinery-loss, NEEDS_PARENT, journal-append-and-commit, dirty-verify and runner-crash rows.
  - id: F11
    severity: MEDIUM
    class: scope
    origin: lens:affected-set
    location: internal/failuretrace/trace.go:332
    summary: The landing failure tracer still loads only Webster's state, so under Weaver every Parent-Align and Finalize verify-failure record loses its card attribution; D13 routes every loom reader of implementer state through the seam, and no card covers this one
    fix: In card 22, add internal/failuretrace#NewTracer, internal/failuretrace#Input and internal/failuretrace/doc.go under Edit. Have NewTracer read the active implementer's runplan.State through weaverengine.Active over the anchor with a nil Setting, returning a seam refusal as its error. Input's state field becomes *runplan.State. Add a row to an existing failuretrace test that traces a failure against a planted Weaver state.
  - id: F12
    severity: MEDIUM
    class: scope
    origin: lens:affected-set
    location: _lym/plan/01-shared-plan-conflict-path.md:13
    summary: Card 1 moves the plan-conflict record but misses part of its affected set — the motif fixture the block-recipe test compares with DeepEqual, conflict.go's file header and websterengine.ScratchDir's comment all still place it under .lym/webster
    fix: In card 1, add internal/loomrecipe/testdata/tapestry-rows-before-motif.yaml, internal/runplan/conflict.go and internal/websterengine#ScratchDir under Edit. Have the Intent say the fixture's Plan-Burler-Review text matches the recipe's new text, the conflict.go header names the implementer-neutral conflict directory, and ScratchDir's comment no longer lists the plan-conflict record or the last-routed conflict.
  - id: F13
    severity: MEDIUM
    class: consistency
    origin: lens:repo-invariants
    location: _lym/plan/09-batch-attempt-strand.md:29
    summary: Weaver adds verify sites (the strand gate closures and runVerify) and reads the card, batch and webster gate keys, but the docs and tripwire that list those sites and keys as Webster's alone stay unchanged
    fix: In card 9, add under Edit pattern/PATTERN-gate-slot.md (its Verified tree site list names Weaver's batch gates and plan-level verify), pattern/PATTERN-test-speed.md (the Gate keys table names Weaver beside Webster for card, batch and webster), internal/gateslot/doc.go (its tier plan names the Weaver row) and cmd/lym#planVerifySites (gains weaverengine's verify files). State each change in the Intent.
  - id: F14
    severity: LOW
    class: consistency
    origin: lens:guard-reach
    location: _lym/plan/23-merge-in-warning.md:30
    summary: The corrupt-Webster row cannot reach its expected warning as written — with the default tapestry.implementer of webster, the advisory fallback reads Webster's corrupt state and emits no warning
    fix: In card 23's TestCommand_MergeScenario sentence, have the corrupt-Webster step set tapestry.implementer to weaver, as card 25's matching row does.
  - id: F15
    severity: LOW
    class: consistency
    origin: lens:guard-reach
    location: _lym/plan/12-run-loop.md:46
    summary: Card 12 prescribes a Stuck naming the re-step when the final commit at Done fails, but no TestRun row reaches it
    fix: In card 12's TestRun paragraph, add a row in which the commit seam fails after the verify passes, so Run returns Stuck naming the failure and the re-step.
  - id: F16
    severity: LOW
    class: consistency
    origin: lens:repo-invariants
    location: _lym/plan/03-weaver-geometry.md:34
    summary: lymdirs/doc.go names UsageDirName and RaddleDirName as the constants under the tracked _lym, and card 3's WeaverDirName adds one without the doc changing
    fix: In card 3, move internal/lymdirs/doc.go from Uses to Edit and say its doc names WeaverDirName beside the tracked-directory exceptions. In card 1, drop internal/lymdirs/doc.go from Uses, since card 3 now targets it.
  - id: F17
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/plan/02-weaver-config.md:43
    summary: Two Extends targets rewrite existing text — card 2 rewrites PATTERN-config-layers' one-line degrading-callers set, and card 20 rewrites loomcli/doc.go's renumber paragraph
    fix: In card 2, move pattern/PATTERN-config-layers.md from Extends to Edit. In card 20, move internal/loomcli/doc.go from Extends to Edit and drop the then-empty Extends group.
  - id: F18
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/plan/22-describe-and-rework-archive.md:34
    summary: The weaver slot constant sits beside the unexported reworkPriorWebster in loomshed, while loomcli's archiveRunRecords must name the same slot, so the slot name would be repeated as a literal (PATTERN-code-conventions)
    fix: In card 22, create plan:internal/loomshed#ReworkPriorWeaver -> const ReworkPriorWeaver = "weaver" in the slot const block. Have archiveRunRecords join loomshed.ReworkPriorWeaver onto dest's parent directory, and have ArchivedRunRecordDirs read the same constant.
---

### [BLOCKING] No-spawn re-entry skips the card gates at HEAD

**Location:** `_lym/plan/13-run-reentry-and-resume.md:33`

**Issue:** D7 says that after any other batch stop, Weaver "runs the full post-batch checks (D11) on the stopped batch against its recorded start commit, with the card gates at HEAD", and records it done with no spawn only if they pass and the tree is clean.
Card 13 runs only the post-batch checks.
`runplan.PostBatchInputs` (`internal/runplan/postbatch.go:102`) carries no gate, so the post-batch checks never run a card gate.
The arbiter's fix can therefore be recorded done without the batch's `card` and `batch` gates ever running over it: the guard this path exists to replace is skipped, and the card states no bound for that skip.

### [BLOCKING] `verify_gate` is a guard key classified CategoryNone

**Location:** `_lym/plan/02-weaver-config.md:34`

**Issue:** PATTERN-config-layers puts a key that "switches off or hollows a guard", an approval gate included, in the Guard category.
That category closes the key to the local override layer.
`verify_gate` names the gate key whose `lym gate test --wait` call is the plan-level verify that the implementer's Done rests on (D10, card 11's `runVerify`).
Classified CategoryNone, a worktree's local file can point it at a narrower key such as `card`, and Weaver's Done then passes on a far weaker verify.
Card 2's Intent also states no bound for the key.

### [BLOCKING] `runplan.ConflictRecordRel` has no production caller

**Location:** `_lym/plan/01-shared-plan-conflict-path.md:45`

**Issue:** `cmd/lym/testonlycode_linux_test.go:61` allowlists `websterengine.ConflictRecordRel` as a test-only oracle.
Card 1 makes it delegate to a new `runplan.ConflictRecordRel`, which no other card calls, and the recipe names the path as static YAML.
The new function is unreachable from any `main` package and not allowlisted, so the test-only-code guard fails card 1's gate.

### [MEDIUM] Card 13 wires `Run` without listing it

**Location:** `_lym/plan/13-run-reentry-and-resume.md:20`

**Issue:** Card 13 says `resumeStrands` and `reenter` are "run by `Run` before it starts anything".
Card 12's `Run` bullets never call them, and card 13 lists `Run` under no group.
The batch that builds card 13 must change a function outside its card's scope, or leave both functions unreached.
The combined overlay check accepts `plan:internal/weaverengine#Run` under card 13's Edit.

### [MEDIUM] `StopRecord` is never cleared

**Location:** `_lym/plan/13-run-reentry-and-resume.md:30`

**Issue:** `reenter` runs whenever `StopRecord` is set (card 13), and card 5 describes the field as the row's last stop.
No card clears it after a successful re-entry or at the next batch begin.
A later plan-conflict Stuck or a crash re-step then reaches `reenter` again over a batch already recorded done.

### [MEDIUM] The fixer's gatefixer record is never marked terminal

**Location:** `_lym/plan/12-run-loop.md:43`

**Issue:** `gatefixer.Await` "never marks rec terminal: the caller does, through its store" (`internal/gatefixer/await.go`).
`SpawnOrAttach` returns a live record without spawning (`internal/gatefixer/spawn.go`).
Card 11's `fixerStore.Live` treats a record with no terminal status as live, and card 12 records only "its outcome".
Unless `Record.Status` is saved through the store, the next fixer and the lost-fixer retry never spawn.

### [MEDIUM] Delivered headings are overwritten by `Run`'s next save

**Location:** `_lym/plan/21-decision-notice-weaver.md:36`

**Issue:** `RecordDelivery` writes into Weaver's state file under the state-mutation lease from the decision verb's process.
`Run` keeps the `*State` it loaded across the whole attempt.
`endAttempt` (card 10, `_lym/plan/10-batch-record.md:33`) saves that copy "under the state-mutation lease", but a lease around the write alone does not stop the stale copy replacing the file.
D13's delivery record is lost, and card 13's resume, which sends only the entries `Delivered` lacks, sends delivered entries again.

### [MEDIUM] Reset step 2's commit is unbounded

**Location:** `_lym/plan/14-weaver-archive-and-reset.md:25`

**Issue:** With no state file, step 2 commits whatever `RecordPending` finds uncommitted under `_lym/weaver` or the plan directory.
That state also holds when no Weaver run ever began, for instance a Webster run or a plan under review.
A stray `lym weaver reset` then commits unrelated plan edits as a finished reset.
D2 does not license this escape, and the card states no bound for it.

### [MEDIUM] Card 21's predicates change signature without re-sign arrows

**Location:** `_lym/plan/21-decision-notice-weaver.md:32`

**Issue:** `decisionNoticeInline(strand string) bool` and `heldBackReason(strand string, halted shedengine.State) string` (`internal/loomcli/decision.go:415,440`) read only their arguments.
The Intent makes a `verify-fixer` strand count as an implementer strand only "while the seam's `ActiveOrSetting` answers Weaver", an input neither function has.
Without the arrows, the `caller-uncovered` check never sees the caller `addedNoticeLines`, which is not listed.

### [MEDIUM] Per-helper tests duplicate TestRun

**Location:** `_lym/plan/09-batch-attempt-strand.md:44`

**Issue:** `classifyAttempt`, `batchGateSpec`, `endAttempt`, `runVerify` and `writeSummary` are unexported helpers of `Run`.
PATTERN-test-speed's test economy tests an unexported helper through the surface that uses it.
TestRun (card 12) already asserts most of their behaviours at `Run`:
- the FAILED, failing-gate and unclean-tree climbs, the machinery-loss retry and the NEEDS_PARENT stop, overlapping TestClassifyAttempt;
- each attempt's journal append and commit, overlapping TestEndAttempt;
- the dirty-tree and runner-crash verify outcomes, overlapping TestRunVerify.

The rows the helper tests alone hold belong in TestRun.

### [MEDIUM] Failure tracer still reads Webster's state only

**Location:** `internal/failuretrace/trace.go:332`

**Issue:** `failuretrace.NewTracer`, which `internal/loomcli/landingdeps.go` wires into Parent-Align and Finalize, calls `websterengine.LoadState` directly.
Under Weaver it finds no state, and every landing verify-failure record loses its card attribution.
D13's opening sentence routes every loom path that reads Webster's run state through the seam, and no card covers this one.

### [MEDIUM] Card 1 misses part of the plan-conflict path's affected set

**Location:** `_lym/plan/01-shared-plan-conflict-path.md:13`

**Issue:** Three places still place the record under the old path:
- `internal/loomrecipe/testdata/tapestry-rows-before-motif.yaml:364` still names `.lym/webster/plan-conflict.md`, and the block-recipe test compares the fixture with the expanded recipe, so the test fails once card 1 changes the recipe text;
- `internal/runplan/conflict.go:2` still places the record in the implementer's scratch directory;
- `websterengine.ScratchDir`'s comment still lists the plan-conflict record and the last-routed conflict.

### [MEDIUM] Verify-site and gate-key docs stay Webster-only

**Location:** `_lym/plan/09-batch-attempt-strand.md:29`

**Issue:** These still describe the sites and keys as Webster's alone:
- `pattern/PATTERN-gate-slot.md`'s Verified-tree site list;
- `pattern/PATTERN-test-speed.md`'s Gate keys table;
- `internal/gateslot/doc.go`'s tier plan;
- `cmd/lym/verifiedtree_test.go`'s `planVerifySites`.

Weaver runs the `card`, `batch` and `webster` calls through `verifytree.Verify` at new sites, and no card updates these.

### [LOW] Card 23's corrupt-Webster row cannot reach its warning

**Location:** `_lym/plan/23-merge-in-warning.md:30`

**Issue:** A corrupt Webster file makes `Active` refuse, and `ActiveOrSetting` falls back to the setting, whose default is `webster`.
`websterengine.RunInFlight` then errors on the corrupt file, and `websterInFlightWarnings` emits nothing (`internal/fabriccli/merge_verbs.go:28-32`).

### [LOW] Done-path commit failure has no reaching test

**Location:** `_lym/plan/12-run-loop.md:46`

**Issue:** The prescribed Stuck on a failed commit of the run record at Done has no TestRun row.
Card 10's commit-failure row covers only the per-batch commit.

### [LOW] lymdirs doc not updated for WeaverDirName

**Location:** `_lym/plan/03-weaver-geometry.md:34`

**Issue:** `internal/lymdirs/doc.go` describes `UsageDirName` and `RaddleDirName` as the exceptions that sit under the tracked `_lym`.
Card 3's `WeaverDirName` joins them, and card 3 lists the doc only under Uses.

### [LOW] Extends targets that rewrite content

**Location:** `_lym/plan/02-weaver-config.md:43`

**Issue:** Adding `weaverengine` inside the brace set on `pattern/PATTERN-config-layers.md:90` rewrites that line.
Card 20's Intent (`_lym/plan/20-renumber-reads-active.md:38`) rewrites `loomcli/doc.go`'s renumber paragraph "in place of webster's".
Extends only adds to a ref.

### [LOW] Weaver's rework slot name would be repeated

**Location:** `_lym/plan/22-describe-and-rework-archive.md:34`

**Issue:** Card 22 adds the `weaver` slot to the unexported const block beside `reworkPriorWebster` in loomshed.
`archiveRunRecords` in loomcli must name the same slot, and it cannot reach that constant.
Two literals of one slot name follow.

## Rejected

- guard-reach F6 (the D1 guard is absent from `webstercli`'s begin-batch and run verbs): D1 places the guard on the implementer row ("the row refuses when the run's other implementer has begun batches"), so a guard on Webster's own verbs is scope the answer key does not license.

## Focus verdicts

- D6's stop-after-record ordering is carried by card 12's `runBatch`, with card 10 recording and committing first. D12's attach and probe steps are carried by card 13; it uses `ProbeGated` in place of `AttachGated` so it can deliver decisions before the wait, as card 9 explains, and the not-found and superseded-strand behaviour is the same. D2's reset refusals are carried by card 14, and D1's corrupt-state repair condition by card 6's `restorableVersion`.
- No card adds a push inside the producer or a gated strand over a fixer commit. Card 11 states the Q18 bound.
- Card order: the one dependency on a change outside its own card's scope is F4.
- Reachability: F1, F5, F6, F8, F14 and F15 cover the gaps found. Every other prescribed refusal names a reaching test row.
- Attack surface: `--stop-strand` is bounded in cards 14 and 15. The seam's advisory fallback is stated in card 6. F2 covers `verify_gate`, and F1 and F8 cover the no-spawn path and the reset's commit branch.
- Test economy: cards 8, 22, 24 and 26 plan no test of stencil or spec prose; their rows assert values Go fills. F10 covers the per-helper tests.
- Typing: F9 covers the missing arrows and F17 the Extends rewrites. Cards 4, 20, 23 and 24 re-sign correctly or change no signature.
- Research steps and board edits: none in any card.
- Verify lines: no card carries a `**Verify:**` line.
- Every proposed fix that adds, drops or moves targets was overlaid together through `lym loom validate-plan --overlay`, with no blocking finding beyond the empty baseline.
