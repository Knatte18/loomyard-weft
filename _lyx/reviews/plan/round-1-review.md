---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    origin: handler
    location: _lyx/plan/03-rebaseline-refusals-keyed.md
    summary: Card 3 keys only the clauses naming lyx webster rebaseline, so auto-path refusals still name lyx webster run and lyx webster rebaseline and card 4's own test cannot pass
  - id: F2
    severity: BLOCKING
    class: scope
    origin: lens:correctness
    location: _lyx/plan/14-fabric-reference-policy-at-record-batch.md
    summary: Card 14 supplies the read-only classifier only through webstercli's runDeps, but the loom's Webster row builds its own RunDeps in loomcli wiring, so D8 stays inert under the loom
  - id: F3
    severity: MEDIUM
    class: scope
    origin: lens:correctness
    location: _lyx/plan/21-one-shell-wait-rule-and-ended-shells.md
    summary: Card 21 removes the shell-expiry state but omits its readers in shuttleengine (shellWaitActive, the held-turn notice) and the Run constructors that must carry the started-fresh fact
  - id: F4
    severity: MEDIUM
    class: scope
    origin: handler
    location: _lyx/plan/19-fabric-way-forward-names-the-orch.md
    summary: Card 19's derived text set omits loomengine LoadConfig, and its claim that fabricengine texts are reached only through a refused verb is false for reader verbs
  - id: F5
    severity: MEDIUM
    class: consistency
    origin: lens:generic
    location: _lyx/plan/11-pattern-entry-line-cap.md
    summary: Card 11 reads card bytes through SourcePath, but the check takes only a Plan, has no root to resolve it against, and Card.Text already holds the raw text
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:test-gaps
    location: _lyx/plan/02-commits-named-by-one-batch-check.md
    summary: Card 2's missing-line row, one-process pin and commit-blob-tree collision row have no construction, since commitsNamedBy calls gitexec directly with no seam
  - id: F7
    severity: MEDIUM
    class: consistency
    origin: lens:test-gaps
    location: _lyx/plan/17-gate-and-verify-drop-the-strand-name.md
    summary: Card 17's TestVerify_Scenario row uses t.Setenv, but that test is parallel and t.Setenv panics under a parallel test or ancestor
  - id: F8
    severity: MEDIUM
    class: design
    origin: lens:error-handling
    location: _lyx/plan/16-fabric-sandboxed-role-guard.md
    summary: The guard in card 16 lands before card 17 stops gate and verify handing tests the strand name, so card 16's own gate run can meet the guard
  - id: F9
    severity: MEDIUM
    class: scope
    origin: lens:correctness
    location: _lyx/plan/04-run-auto-rebaseline.md
    summary: Card 4 says the rebaseline warning reaches the run summary but names no mechanism, since finishMasterDone puts entry warnings only on RunResult.Warnings
  - id: F10
    severity: MEDIUM
    class: consistency
    origin: lens:error-handling
    location: _lyx/plan/05-webster-row-auto-rebaseline-stuck.md
    summary: Card 5 claims the ErrAutoRebaseline text already names re-stepping, but SaveState and restampBaseline failures return bare errors and no card adds the clause or a test row
  - id: F11
    severity: MEDIUM
    class: consistency
    origin: lens:error-handling
    location: _lyx/plan/15-recover-batch-accepts-read-only-fabric-references.md
    summary: Card 15 passes deps.ReadOnly into AcceptBatchFabricReference without the nil-accepts-nothing rule card 14 states, so an unset classifier would panic or leave the in-line precheck undefined
  - id: F12
    severity: MEDIUM
    class: scope
    origin: lens:test-gaps
    location: _lyx/plan/23-recovery-count-and-retry.md
    summary: The real-git side of RecoveryRetry (nonMergeCommitsBetween, first-parent range, merge-only range, absent start SHA) is tested only over the git fake
  - id: F13
    severity: MEDIUM
    class: scope
    origin: lens:test-gaps
    location: _lyx/plan/09-plan-review-webster-record-marker.md
    summary: Nothing pins the Plan-Burler fixer path D2 exists to serve, since no test shows burlerRoundProfile's rubric or the Bouncer seed and judge calls carrying the webster_record note
  - id: F14
    severity: MEDIUM
    class: design
    origin: lens:generic
    location: _lyx/plan/13-read-only-classifier-allow-list.md
    summary: D8 says the classifier admits a subset of the guard's readers built from the same verb names, but cards 13 and 16 hard-code separate lists with no shared declaration or subset pin
  - id: F15
    severity: LOW
    class: scope
    origin: lens:error-handling
    location: _lyx/plan/04-run-auto-rebaseline.md
    summary: The rebaseline warning prefix is applied inside finishMasterDone only, so a verify-gate demotion of StuckReason and error-return exits after an accepted rebaseline can drop it
  - id: F16
    severity: LOW
    class: design
    origin: lens:generic
    location: _lyx/plan/06-edited-done-cards.md
    summary: TestEditedDoneCards re-covers the cases card 7 pins through TestNewPlanGate, while the begun-but-not-done and unreadable-file rows the checks invite are missing
  - id: F17
    severity: LOW
    class: scope
    origin: lens:test-gaps
    location: _lyx/plan/23-recovery-count-and-retry.md
    summary: Acceptance says a rebaseline keeps the recovery count, but no card adds a row asserting it
  - id: F18
    severity: LOW
    class: scope
    origin: lens:generic
    location: _lyx/plan/16-fabric-sandboxed-role-guard.md
    summary: Three acceptance clauses have no assertion, namely the add long help stating it pushes to origin, lyx config's per-worktree write syncing under a webster strand name, and the implementer stencil stating the sandbox rule
  - id: F19
    severity: LOW
    class: design
    origin: lens:generic
    location: _lyx/plan/15-recover-batch-accepts-read-only-fabric-references.md
    summary: Cards 15 and 16 do not state their attack-surface bound, namely the evidence rule bounding in-line accept and what a typed bypass push lets through
  - id: F20
    severity: LOW
    class: decision
    origin: lens:error-handling
    location: _lyx/plan/23-recovery-count-and-retry.md
    summary: Card 23 leaves open what recoverSpawn does when reading HEAD for RecoveryStartSHA fails, where an empty start silently disables the retry
  - id: F21
    severity: LOW
    class: scope
    origin: lens:generic
    location: _lyx/plan/03-rebaseline-refusals-keyed.md
    summary: Cards 3 and 6 change observable websterengine behaviour but list no doc.go edit, against the overview's own same-commit decision
  - id: F22
    severity: LOW
    class: scope
    origin: lens:test-gaps
    location: _lyx/plan/13-read-only-classifier-allow-list.md
    summary: Test rows are thin in several cards, with no accepted row per git form in card 13, no cap-boundary or multibyte row in card 11, one gate row for three runTest branches in card 17, and no shuttle-side log assertion in card 21
  - id: F23
    severity: NIT
    class: consistency
    origin: lens:generic
    location: _lyx/plan/19-fabric-way-forward-names-the-orch.md
    summary: ImpactSummary lines pin perishable tallies in cards 9 and 19, card 19 lists the hubgeom test with no stated change, and card 16 lists an unexported configcli symbol under Uses
---

### [BLOCKING] Card 3 leaves auto-path refusals naming `lyx webster run` and `lyx webster rebaseline`

**Location:** `_lyx/plan/03-rebaseline-refusals-keyed.md`, `_lyx/plan/04-run-auto-rebaseline.md`, `_lyx/plan/00-overview.md` (Shared Decision `step-keyed-refusal-texts`)

**Issue:** D1's acceptance rows say the done-card refusal names neither `lyx webster run` nor `lyx webster rebaseline`, and no auto-path refusal contains `lyx webster rebaseline`.
Card 3 keys only three clause kinds: the follow-up landing, the unnamed-card clause and the transient retries.
In `internal/websterengine/rebaseline.go`, four more refusal sites end in `freshRestartSteps(stepRun)`, which renders `lyx webster reset --to start` then `lyx webster run`: the no-frame `overviewRefusal` (line 69), the begun-batch card-set change in `rebaselineBatches` (line 152), and both branches of the changed and done-changed way forward (lines 295 and 300).
`rebaselineBatches` also calls `CheckBatchOrder` (line 162), whose way forward at `sequence.go:37` is hard-coded to `lyx webster rebaseline --card NN` and `lyx webster run --fresh`; a badly ordered follow-up card is the main D2 scenario, and card 4 wraps that error in `ErrAutoRebaseline`.
`CheckBatchOrder` is in no card's targets.
Card 4's `TestRun_AutoRebaseline` asserts the done-card text names neither verb, and card 3 asserts only the absence of `lyx webster rebaseline`, so the plan's own test cannot pass with the cards' stated scope, and the Shared Decision overclaims.

**Fix:** In card 3, key every `freshRestartSteps(stepRun)` site on `RebaselineDeps.Step` (`resetToStartSteps` already takes a re-entry argument), and add `internal/websterengine#CheckBatchOrder` with a re-sign arrow taking the step (its callers in `partition.go` and `rebaselineBatches` pass `stepRun` or the deps' step) plus its test.
Extend the step-set rows in `TestRebaseline_CardSet` to the mixed done-and-card-set case, and have card 4 assert both verbs absent on every `ErrAutoRebaseline` row, including an out-of-order follow-up card.

### [BLOCKING] Card 14 leaves D8 inert under the loom

**Location:** `_lyx/plan/14-fabric-reference-policy-at-record-batch.md`, with `internal/loomcli/wiring.go:476`

**Issue:** Card 14 adds `RunDeps.ReadOnly` and supplies it only from `webstercli`'s `runDeps`.
The loom's Webster row builds its own `websterengine.RunDeps` literal in `loomCLI.wire` (`internal/loomcli/wiring.go:476`), which stays nil.
Under the loom, `runExitAuditCrossCheck` then classes every read-only fabric reference as correctness, the halt D8 exists to remove, and a nil classifier accepts nothing by the card's own rule.

**Fix:** Add `internal/loomcli#loomCLI.wire` and `internal/loomcli/wiring_test.go` (a row asserting the closure is set) to card 14's `Edit:`, or have both sites share one constructor.

### [MEDIUM] Card 21 omits the readers and constructors the rewrite touches

**Location:** `_lyx/plan/21-one-shell-wait-rule-and-ended-shells.md`, with `internal/shuttleengine/waitmark.go:173`, `hold.go:46`, `run.go:505`, `attach.go`

**Issue:** `Run.shellWaitActive` in `waitmark.go` reads `run.expiredShells` and "not yet waited out", state the card removes; it and its tests in `waiting_test.go` are in no target.
The started-fresh fact must be set where a `Run` is built, `Runner.Start` and the `Attach` family, none of which is a target.
After the rewrite `expiredTurnEnd` never returns a held turn end carrying tasks, leaving `heldTurnEnd.tasks` and the task branch of `holdNotice` dead with no card deleting them.
The wait marker is covered by PATTERN-wait-mark-display-only, so its change needs a card.

**Fix:** Add `internal/shuttleengine#Run.shellWaitActive`, `internal/shuttleengine#Runner.Start`, the Attach constructors, `internal/shuttleengine#heldTurnEnd` and `holdNotice` (or state the dead field is kept) and their tests to card 21, and say what the marker shows once nothing expires.

### [MEDIUM] Card 19's derived set is incomplete and its exemption premise is false

**Location:** `_lyx/plan/19-fabric-way-forward-names-the-orch.md`, with `internal/loomengine/config.go:466`, `internal/fabricengine/origin.go:109`

**Issue:** `loomengine.LoadConfig` emits `not initialized here; run "lyx fabric reconcile"`, the twin of the `landingshed` and `boardengine` texts the card rewords; its exact-string `TestLoadConfig_NotInitialized` is at `loomengine/config_test.go:360`.
Loom is inside the card's own search domain.
The card exempts `fabricengine` texts because a refused session reaches them only through a refused verb first; that fails for `fabricengine.LoadConfig`, which the reader verbs `list`, `pairs` and the dry-run `prune` and `cleanup` call, and for `origin.go:109`, reached through `hubgeom.ResolveParent` from `webster reset`.
`reconcile` is refused by the guard, so those texts send a refused session to a verb it cannot run.

**Fix:** Add `internal/loomengine#LoadConfig` and `internal/loomengine/config_test.go`, reword the `fabricengine` texts that a reader verb or `hubgeom` reaches (or state why each is exempt), and re-run the search over the Go error texts.

### [MEDIUM] Card 11 reads card bytes through a path it cannot resolve

**Location:** `_lyx/plan/11-pattern-entry-line-cap.md`, with `internal/planparser/plan.go:111`

**Issue:** `Card.SourcePath` is the worktree-relative token `_lyx/plan/NN-slug.md`, and `checkPatternEntryLineCap(plan *Plan)` has no root to resolve it against; a disk read also adds an I/O error path to an otherwise pure check.
`Card.Text` already holds the raw card text, and its field comment says only `CheckCardFabricReference` reads it.
`planparser/doc.go` says the package reads no file.

**Fix:** Scan `Card.Text`, drop the `SourcePath` read, and update the `Card.Text` comment to name the new reader.

### [MEDIUM] Card 2's test rows have no construction

**Location:** `_lyx/plan/02-commits-named-by-one-batch-check.md`, with `internal/websterengine/gitwrap.go:464`

**Issue:** `commitsNamedBy` is a free function over `gitexec`, so a fake-fed `missing` line has no seam, and real git cannot report `missing` for an object `rev-parse` just listed.
`TestRepositoryProbes` runs real git with no call counter, so "counting the probe's git calls where the fixture already can, or asserting the output shape otherwise" leaves the choice open.
A prefix shared by a commit, a blob and a tree needs a crafted collision the card does not describe.

**Fix:** Create a small pure parser for the batch-check output and table-test the `missing` and malformed rows through it.
Name how the collision fixture is built, or narrow the row to something constructible, and drop or specify the process-count claim.

### [MEDIUM] Card 17's verify row cannot use `t.Setenv`

**Location:** `_lyx/plan/17-gate-and-verify-drop-the-strand-name.md`, with `internal/verifytree/verify_integration_test.go:134`

**Issue:** `TestVerify_Scenario` calls `t.Parallel()`, and `t.Setenv` panics in a test whose own or any ancestor's `t` is parallel, so the planned row panics.

**Fix:** Drive `Verify` under the strand name through a re-executed helper process, or put the row in a non-parallel top-level test that names the global state and raise the `verifytree` budget row.

### [MEDIUM] Card 16 lands before card 17

**Location:** `_lyx/plan/16-fabric-sandboxed-role-guard.md`, `_lyx/plan/17-gate-and-verify-drop-the-strand-name.md`

**Issue:** The guard lands in card 16, but the drop of the strand name from gate and verify runs lands in card 17.
An implementer's `lyx gate test` for card 16 and every later card runs under a slugged strand name, and any test that spawns `lyx` and drives a refused fabric verb meets the guard in the span between the two cards, which is the order risk D7 closes.
Card 17 depends on nothing card 16 creates.

**Fix:** Order the gate and verify env drop before the guard.

### [MEDIUM] Card 4 names no path from the rebaseline warning to the run summary

**Location:** `_lyx/plan/04-run-auto-rebaseline.md`, with `internal/websterengine/runlevel.go:755`

**Issue:** The card says the warning lands "in the summary's warnings", but `finishMasterDone` puts entry warnings only on `RunResult.Warnings`.
The only warnings that reach `summary.md` are the recorded audit warnings, through `appendRecordedAuditWarnings`, and the shed adapter's Done outcome points at `summary.md`, never at `RunResult.Warnings`.
D1's acceptance row "the run summary carries a warning naming the rebaseline" cannot hold as written.

**Fix:** State the mechanism (for example `finishMasterDone` appending the warning to the summary) and add the matching assertion to `TestRun_AutoRebaseline`.

### [MEDIUM] Card 5's re-step claim is false for save and restamp failures

**Location:** `_lyx/plan/05-webster-row-auto-rebaseline-stuck.md`, `_lyx/plan/04-run-auto-rebaseline.md`, with `internal/websterengine/fingerprint.go:216`

**Issue:** Card 4 wraps every error in `ErrAutoRebaseline` "with the refusal's or the transient's own text", and card 5 says the text already names re-stepping the row.
`restampBaseline` returns bare errors from its fingerprint, plan-file-hash and baseline-store calls, and `SaveState` errors are bare too; D1 requires a transient's text to say to re-step the row.
Card 3 keys only three `Rebaseline` transient sites, and no test row checks the text of a save or restamp failure.

**Fix:** State in card 4 that `autoRebaseline` appends the re-step clause to any non-refusal error, add rows for a `SaveState` failure and a `restampBaseline` failure to `TestRun_AutoRebaseline`, and add step-set rows for the transient clauses at `rebaseline.go:203` and `:265`.

### [MEDIUM] Card 15 lacks the nil-classifier rule

**Location:** `_lyx/plan/15-recover-batch-accepts-read-only-fabric-references.md`, with `internal/websterengine/auditledger.go:239`

**Issue:** `AcceptBatchFabricReference` calls `readOnly(cmd)` unguarded, and card 15 passes `deps.ReadOnly`, a new optional `RecoverDeps` field, into it.
Existing `TestRecoverSpawnOrAttach` rows build deps without it.
Card 14 says a nil classifier accepts nothing, but card 15's in-line precheck is undefined for nil.

**Fix:** State that a nil `ReadOnly` skips the in-line route and leaves `needs_fresh` unchanged, with a row for it.

### [MEDIUM] Card 23's `RecoveryRetry` git side is untested against git

**Location:** `_lyx/plan/23-recovery-count-and-retry.md`

**Issue:** `TestRecoveryRetry` runs only over the git fake, so `realGit.NonMergeCommitsBetween` and `nonMergeCommitsBetween` (first-parent range, merge-in-only range, start SHA absent from the store) never run against a repository, and the card lists no `gitwrap_test.go`.

**Fix:** Add rows to `TestRepositoryProbes` over its scratch repo: commits past a start, a merge-only range, and an unknown SHA returning an error.

### [MEDIUM] Card 9 does not test the path D2 serves

**Location:** `_lyx/plan/09-plan-review-webster-record-marker.md`

**Issue:** The fixer sees the record through `burlerRoundEntry`, `burlerRoundProfile` and `ReadRubric`, but the card edits no burler entry test, and `TestBouncerEntry_HappyPath` and `TestReadRubric` do not show the fixer's explore rubric or the Bouncer seed and judge calls carrying the note.

**Fix:** Add a row, in the existing test for `burlerRoundEntry`, asserting the profile's rubric contains the `webster_record` text from `Env.WebsterRecord`, and a Bouncer row for the seed and judge rubric.

### [MEDIUM] Cards 13 and 16 can drift on the reader set

**Location:** `_lyx/plan/13-read-only-classifier-allow-list.md`, `_lyx/plan/16-fabric-sandboxed-role-guard.md`

**Issue:** D8 says the classifier admits a subset of the guard's readers, built from the same verb names.
Card 13 hard-codes `list`, `pairs`, `status`, `shortname`, `diff`, `prune` and `cleanup`; card 16's `readerVerb` hard-codes its own list; nothing shares them or pins the subset.

**Fix:** Have card 13 create one reader-verb declaration in `fabricengine`, card 16 list it under `Uses:` and build `readerVerb` from it, and add a row to card 16's subcommand walk that every verb the classifier admits is a guard reader.

### [LOW] Card 4's warning can be dropped on some Stuck exits

**Location:** `_lyx/plan/04-run-auto-rebaseline.md`, with `internal/websterengine/runlevel.go:526`, `:773`

**Issue:** The prefix is applied in `finishMasterDone` only.
The verify-gate demotion at lines 773 to 776 overwrites `StuckReason`, and error returns such as `pendingAuditFindingsError` (mapped to Stuck by the adapter) carry no warning; D1 says a stuck outcome's reason opens with it.

**Fix:** Apply the prefix after the demotion, say what the error-return exits do (log the warning, as the `--fresh` drop does, or prefix the reason), and add a row for the demoted case.

### [LOW] Card 6's test duplicates card 7's and misses invited rows

**Location:** `_lyx/plan/06-edited-done-cards.md`, `_lyx/plan/07-plan-gate-done-card-edited.md`

**Issue:** `TestEditedDoneCards` pins the five cases card 7 pins through `TestNewPlanGate`, and `EditedDoneCards` has no other consumer.
No row shows an edited card of a begun but not-done batch reporting nothing, or the unreadable-file error.

**Fix:** Fold the shared cases into the gate rows, keep only the unreadable-file and nil-state rows, add the begun-not-done row, and drop the extra top-level test and its budget bump.

### [LOW] No row shows a rebaseline keeping the recovery count

**Location:** `_lyx/plan/23-recovery-count-and-retry.md`

**Issue:** D12's acceptance says a re-begin and a rebaseline keep the count; card 23 pins the re-begin only.

**Fix:** Add a `Recoveries` assertion to an existing `TestRebaseline_*` row.

### [LOW] Card 16 leaves acceptance clauses unasserted

**Location:** `_lyx/plan/16-fabric-sandboxed-role-guard.md`, `_lyx/plan/18-implementer-sandbox-rule.md`

**Issue:** No assertion that `add`'s long help says it pushes to origin (`help_test.go` already pins verb declarations).
The acceptance row that `lyx config`'s per-worktree write syncs under a webster strand name is tested only through `fabriccli.RunCLIIn`, not through `configcli`.
Nothing asserts the implementer stencil states the sandbox rule.

**Fix:** Assert the help substring in `help_test.go`, add a webster-strand row to `configcli`'s guard table, and cover the stencil through an existing stencil content test, listing each file.

### [LOW] Cards 15 and 16 do not state their attack-surface bound

**Location:** `_lyx/plan/15-recover-batch-accepts-read-only-fabric-references.md`, `_lyx/plan/16-fabric-sandboxed-role-guard.md`

**Issue:** Card 15 lets Master clear a correctness-class audit halt in line, and its Intent never names what bounds that: the classifier's static shape plus the evidence rule (HEAD at the batch start, or the start an ancestor and the tree clean).
Card 16 says a bypass-flag `push` passes without stating that a typed bypass `push` pushes the branches of the worktree paths it names.

**Fix:** Add one `Bound:` sentence to each card.

### [LOW] Card 23 leaves a HEAD-read failure undecided

**Location:** `_lyx/plan/23-recovery-count-and-retry.md`

**Issue:** `recoverSpawn` reads HEAD into `RecoveryStartSHA` before `Starter.Start`, but a failed read is unspecified; recording an empty start silently disables the retry, because D12 treats an empty start as false.

**Fix:** Decide it in the Intent (return the error with a way forward, since nothing has started) and add a `TestRecoverSpawnOrAttach` row.

### [LOW] Cards 3 and 6 list no `doc.go` edit

**Location:** `_lyx/plan/03-rebaseline-refusals-keyed.md`, `_lyx/plan/06-edited-done-cards.md`

**Issue:** The overview's `refusal-and-doc-in-the-same-commit` decision requires the module's `doc.go` passage for a changed observable behaviour.
Card 3 changes the logged cause, the no-frame refusal and the step keying, which `internal/websterengine/doc.go` describes; card 6 adds `EditedDoneCards` beside `DoneCards`, which it also describes.

**Fix:** Add `internal/websterengine/doc.go` to both cards' `Edit:`.

### [LOW] Test rows are thin in four cards

**Location:** `_lyx/plan/13-read-only-classifier-allow-list.md`, `_lyx/plan/11-pattern-entry-line-cap.md`, `_lyx/plan/17-gate-and-verify-drop-the-strand-name.md`, `_lyx/plan/21-one-shell-wait-rule-and-ended-shells.md`

**Issue:** Card 13's accepted examples cover few git forms, so a regression that over-rejects (`log -n 5`, `log -5`, `status -b`, `branch --list pat`, `remote show -n o`, `config --get-regexp`, the `--` path split) passes.
Card 11 has no row at exactly the cap versus one over or for a multibyte rune, though the check counts runes.
Card 17 claims the slotted, inherited-slot and unslotted branches of `runTest` but adds one gate row.
Card 21 attributes the end-of-run log assertion to websterengine tests, which cannot see shuttle's log line.
Card 4's "option unset returns `ErrFingerprintMismatch`" rows repeat `TestRun_RefusesBeforeSpawn`, and its in-flight amendment row skips the "next recovery's prompt renders it" half of the acceptance.

**Fix:** Add one accepted row per git form, boundary and non-ASCII rows, a row per `runTest` branch (or build the stripped environment once before the branches), the log assertion in shuttleengine's rows, move card 4's duplicates into the existing test, and add the prompt-render assertion.

### [NIT] Perishable tallies and stray targets

**Location:** `_lyx/plan/19-fabric-way-forward-names-the-orch.md`, `_lyx/plan/09-plan-review-webster-record-marker.md`, `_lyx/plan/16-fabric-sandboxed-role-guard.md`

**Issue:** Card 19 pins "Thirteen error texts across eight modules" (the targets span nine packages) and card 9 pins "four callers" (the card itself targets another call site).
Card 19 lists `internal/hubgeom/webstergeom_test.go` under `Edit:` though its substring assertion survives the clause, and its ImpactSummary says that test asserts a text `fabricengine` owns, while `webstergeom_test.go:106` asserts the `gateslot.ErrConfigAbsent` text the card rewords.
Card 16 lists the unexported `configcli#hubWriteRefusal` under `Uses:` as a model, which no code in `fabriccli` can use.

**Fix:** State the conclusion without counts, drop the `hubgeom` target or the claim, and drop the `Uses:` line.

## Rejected

- Card 21 should be split into a shuttle wait-rule card and a record-and-consumers card (lens:generic): the rename of `ExpiredShells` must stay with its consumers, and the wait rule and the record share `finalize` and the reversed tests, so the split gains no independently reviewable unit.
- Card 8 renders `none` for an unreadable webster record while card 7 returns an error (lens:error-handling): follows the `PublishFailureNote` precedent the card cites, and a rubric read must never fail.
- Card 15 or 14 attack-surface findings for cards 4, 13, 14, 17, 21, 23 and 24 (lens:error-handling and lens:generic): each states what it lets through and its bound, so the attack-surface question is answered.
- Cards 6 and 8 split from their consumers (focus directive, granularity): `EditedDoneCards` and `WebsterRecordNote` are exported surfaces reused across packages, which the rubric gives their own cards.
- Splitchain edge-case rows (lens:test-gaps NIT, unterminated quote, CRLF continuation, trailing separator): the record names the rows it wants, and rubric test economy bars cataloguing further cases.
