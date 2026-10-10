---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    origin: lens:correctness
    location: _lyx/plan/04-run-auto-rebaseline.md (Intent, error wrapping)
    summary: A failure of MerriamBase inside autoRebaseline is not wrapped in ErrAutoRebaseline, so the Webster row fails hard instead of blocking Stuck
  - id: F2
    severity: MEDIUM
    class: scope
    origin: lens:test-gaps
    location: _lyx/plan/17-gate-and-verify-drop-the-strand-name.md (Intent, TestVerify_DropsStrandName)
    summary: Only the unslotted Verify branch is pinned, so the slotted branch can keep handing tests the strand name
  - id: F3
    severity: MEDIUM
    class: scope
    origin: lens:test-gaps
    location: _lyx/plan/14-read-only-classifier-allow-list.md (TestIsReadOnlyCommand rows)
    summary: No rejected row for the glob characters or a leading tilde that plainWord must refuse, and no accepted row for a hyphenated lyx help command
  - id: F4
    severity: LOW
    class: scope
    origin: handler
    location: _lyx/plan/04-run-auto-rebaseline.md (Intent, paragraphs on Run's later refusals)
    summary: Card wraps Run's later validation, zero-batch and quarry refusals in ErrAutoRebaseline and prefixes every later error with the warning, which D1 does not license
  - id: F5
    severity: LOW
    class: consistency
    origin: lens:correctness
    location: _lyx/plan/03-rebaseline-refusals-keyed.md (Intent, CheckBatchOrder step)
    summary: The no-partition branch of rebaselineBatches reaches CheckBatchOrder through formBatches with the empty step, so an auto-path refusal still names the manual verbs
  - id: F6
    severity: LOW
    class: decision
    origin: lens:correctness
    location: _lyx/plan/22-one-shell-wait-rule-and-ended-shells.md (Intent, started-fresh field)
    summary: The started-fresh field leaves open whether a run that resumes a session counts as fresh, and the Edit list names Runner.attach where the attach Run is built elsewhere
  - id: F7
    severity: LOW
    class: consistency
    origin: lens:error-handling
    location: _lyx/plan/24-recovery-count-and-retry.md (Intent, recoverSpawn HEAD read)
    summary: The new pre-Start HEAD read does not say it replaces the existing post-Start HEAD read, which can still fail after a strand has started unrecorded
  - id: F8
    severity: LOW
    class: scope
    origin: lens:test-gaps
    location: _lyx/plan/02-commits-named-by-one-batch-check.md (test paragraph)
    summary: No planned row fails if commitsNamedBy returns to one cat-file call per object, though the acceptance row is about the single batch-check
  - id: F9
    severity: LOW
    class: consistency
    origin: lens:test-gaps
    location: _lyx/plan/03-rebaseline-refusals-keyed.md (TestRebaseline_OverviewFrameUnreadable paragraph)
    summary: An existing TestRebaseline_EditedPlan row asserts the old no-frame text the card rewords, and the card puts the new absent-copy assertions in a different test
  - id: F10
    severity: LOW
    class: design
    origin: lens:test-gaps
    location: _lyx/plan/06-edited-done-cards.md and _lyx/plan/07-plan-gate-done-card-edited.md
    summary: The unreadable card file case is pinned twice, once in TestEditedDoneCards and once as a TestNewPlanGate row
  - id: F11
    severity: LOW
    class: scope
    origin: lens:test-gaps
    location: _lyx/plan/18-fabric-sandboxed-role-guard.md (TestSandboxedRoleGuard rows)
    summary: No row asserts the refusal text names the hubforge fixture and status FAILED way forward, and --help passing is pinned for add only
  - id: F12
    severity: LOW
    class: scope
    origin: lens:test-gaps
    location: _lyx/plan/04-run-auto-rebaseline.md (TestRun_AutoRebaseline rows)
    summary: The SaveState failure row has no stated forcing mechanism, no row asserts Run starts nothing after a failed save, and the Warn and Info line rows do not assert exactly one line
  - id: F13
    severity: LOW
    class: scope
    origin: lens:test-gaps
    location: _lyx/plan/24-recovery-count-and-retry.md (TestRecoveryRetry rows)
    summary: No row for a terminal prior that is not dead, such as stuck or failed, with commits past its start
  - id: F14
    severity: NIT
    class: consistency
    origin: lens:generic
    location: _lyx/plan/01-gitexec-run-stdin.md (Intent, doc.go sentence)
    summary: RunStdin is called the third checked form beside Run and RunGit, but RunGit is the raw form so RunStdin is the second checked form
  - id: F15
    severity: NIT
    class: consistency
    origin: lens:generic
    location: _lyx/plan/18-fabric-sandboxed-role-guard.md (Edit list)
    summary: The Intent says the package comment at the top of fabric.go names the guard, but fabric.go is not a target
---

### [MEDIUM] MerriamBase failure in autoRebaseline escapes the Stuck mapping

**Location:** `_lyx/plan/04-run-auto-rebaseline.md`, Intent.
**Issue:** The card has `autoRebaseline` compute Merriam's base and wraps in `ErrAutoRebaseline` only the `Rebaseline` errors, the `SaveState` failure and the `changedPlanFiles` read failure.
`MerriamBase` (`internal/websterengine/merriambase.go`) reads `CLAUDE.md`, the Master stencil, `PATTERN.md` and `00-overview.md` and fails with `merriamBaseError`, a transient.
Unwrapped, that error reaches `WebsterProducer.Call` as a returned error and fails the run with `producer-hard-failure`.
D1 says a transient read failure blocks the row as Stuck, with text that says to re-step the row, and its acceptance row says a transient rebaseline failure blocks it as Stuck.
**Fix:** Add the `MerriamBase` failure to the wrapped set with the re-step clause, and add a `TestRun_AutoRebaseline` row that forces it, for example with an unreadable `00-overview.md`.

### [MEDIUM] Verify's slotted branch is unpinned

**Location:** `_lyx/plan/17-gate-and-verify-drop-the-strand-name.md`, Intent.
**Issue:** `Verify` builds its environment in two branches: `lease.Env(os.Environ())` when slots are held, and nil when unslotted.
The card changes both, but `TestVerify_DropsStrandName` covers the unslotted run only.
The slotted branch is the production route under `PATTERN-gate-slot`, and a regression that keeps the strand name there passes every planned row, which breaks D7's guarantee that no verify run hands a test the strand name.
**Fix:** Add the strand name and a variable-absent assertion to `TestVerify_SlotGate`'s env-printing command so the slotted branch is pinned, or give the new test a slotted row.

### [MEDIUM] Classifier test rows miss plainWord's glob and tilde classes

**Location:** `_lyx/plan/14-read-only-classifier-allow-list.md`, `TestIsReadOnlyCommand` paragraph.
**Issue:** D8 makes `plainWord` refuse `*`, `?`, `[` and a leading `~` as well as quote, backslash, `$` and brace.
The card's rejected rows are the record's list, which covers quote, backslash, `$` and brace shapes but no glob character or leading `~`.
A `plainWord` that forgot those characters passes every listed row, and the record circled four review rounds on bypass shapes of this classifier.
The `lyx` help form says "lowercase command words" and the only help rows are `lyx fabric add --help` and one nested word, so an implementation reading `[a-z]+` would reject `lyx webster record-batch --help` and still pass every row.
**Fix:** Add one rejected row per character class (`git log *`, `git log a?`, `git log [a]`, `git log ~/x`), a rejected uppercase help word, and one accepted hyphenated help row.

### [LOW] Card 4 widens ErrAutoRebaseline past what D1 licenses

**Location:** `_lyx/plan/04-run-auto-rebaseline.md`, the paragraphs on `Run`'s own refusals after an accepted rebaseline.
**Issue:** D1 wraps "every error the auto-rebaseline and its save return".
The card also rewords and wraps `Run`'s later plan-validation, zero-batch and quarry-transient refusals, re-signs `zeroBatchesError` for them, and prefixes every error `Run` returns after an accepted rebaseline with the warning.
Those errors are returned today and become Stuck under the card, a disposition the record never states; its acceptance rows need only the warning on a Stuck reason and in the summary.
**Fix:** Cut the extension and the `zeroBatchesError` re-sign, or name in the Intent the record clause that licenses it and its bound, keeping the test rows.

### [LOW] No-partition branch still prints manual verbs on the auto path

**Location:** `_lyx/plan/03-rebaseline-refusals-keyed.md`, Intent on `CheckBatchOrder`.
**Issue:** `rebaselineBatches` takes its no-partition branch through `formBatches`, which calls `CheckBatchOrder`, and the card has `formBatches` pass the empty step.
An order violation on a record without a partition therefore names `lyx webster rebaseline --card NN` and `lyx webster run --fresh` under the loom, against the acceptance row that no auto-path refusal contains `lyx webster rebaseline`.
**Fix:** Check the order in that branch with the deps' step, or pass the step into `formBatches` for that call, and add the row to `TestRebaseline_CardSet`.

### [LOW] Started-fresh field leaves the resumed-session case open

**Location:** `_lyx/plan/22-one-shell-wait-rule-and-ended-shells.md`, Intent on the started-fresh field.
**Issue:** D9 counts at once only for a run "started fresh rather than attached or resumed".
`Runner.start` also launches a run with a `ResumeSessionID` (it calls `checkResume`), whose output files can predate the run, the hazard the exception guards against; the card does not say whether that run counts.
The card also lists `Runner.attach` as a builder, but the attached `Run` is built in `reconstruct` (`attach.go`), which leaves the field unset without any edit to `attach`.
**Fix:** State that `Runner.start` sets the field only when `ResumeSessionID` is empty, and drop `Runner.attach` from the Edit list.

### [LOW] Two HEAD reads in recoverSpawn

**Location:** `_lyx/plan/24-recovery-count-and-retry.md`, Intent on `recoverSpawn`.
**Issue:** `recoverSpawn` reads HEAD after `Starter.Start` and `FindRun` as the fallback `start` value and returns that read's error bare.
The card adds an earlier read for `RecoveryStartSHA` with a transient way forward and says nothing about the later one.
If both stay, a strand can still start, fail on the second read, and be left unrecorded and uncounted, so the row "a failed HEAD read starts nothing" passes while that window stays open.
**Fix:** Say that the single pre-`Start` read supplies both `RecoveryStartSHA` and the `start` fallback and that the post-`Start` `HeadSHA` call is removed.

### [LOW] Nothing pins the single batch-check call

**Location:** `_lyx/plan/02-commits-named-by-one-batch-check.md`, test paragraph.
**Issue:** The acceptance row is that `commitsNamedBy` runs one batch-check.
Every planned row asserts returned values, so a per-object `cat-file -t` loop passes them all.
**Fix:** Pin the call count in the existing `commitsNamedBy` subtest, for example with a recording `git` earlier on `PATH`, or state in the card that the single call is pinned only by `RunStdin` being the sole call path.

### [LOW] Existing rebaseline row asserts the reworded text

**Location:** `_lyx/plan/03-rebaseline-refusals-keyed.md`, `TestRebaseline_OverviewFrameUnreadable` paragraph.
**Issue:** `TestRebaseline_EditedPlan` has a row, a state without the overview frame hash and without the baseline copy, that already reaches the absent-copy refusal and asserts `recorded no overview frame` and `1) lyx webster reset --to start; 2) lyx webster run`.
The card rewords that text but names neither the row nor its update, and puts the absent-copy cause, path and log assertions into `TestRebaseline_OverviewFrameUnreadable`, a test of the transient-after-parse path.
**Fix:** Put the absent-copy assertions on that existing row, state that its `wantText` changes, and add new rows only for the two causes no existing test reaches.

### [LOW] Unreadable-card case pinned twice

**Location:** `_lyx/plan/06-edited-done-cards.md` and `_lyx/plan/07-plan-gate-done-card-edited.md`.
**Issue:** An unreadable card file returning an error is a row of the new `TestEditedDoneCards` and a row of `TestNewPlanGate`, and the gate row exercises `EditedDoneCards` through `ValidatePlan`.
`PATTERN-test-economy` holds one test per behavior.
**Fix:** Keep the case in one place, dropping the `TestNewPlanGate` row or the `TestEditedDoneCards` row.

### [LOW] Guard way-forward text and help rows unasserted

**Location:** `_lyx/plan/18-fabric-sandboxed-role-guard.md`, `TestSandboxedRoleGuard` paragraph.
**Issue:** The rows assert that a verb refuses or runs, never that the refusal names the `hubforge` fixture and the `status: FAILED` step, which D7 requires of the refusal's way forward.
`--help` passing under a refused role is pinned for `add` only, though the guard's reader rule covers every `--help`.
**Fix:** Assert both way-forward fragments on one refusal row, and run `--help` for every walked subcommand.

### [LOW] Auto-rebaseline save failure and log-once rows underspecified

**Location:** `_lyx/plan/04-run-auto-rebaseline.md`, `TestRun_AutoRebaseline` paragraph.
**Issue:** The file in place of the plan-baseline directory forces the restamp failure; the card names no way to force a `SaveState` failure.
No row asserts that `Run` starts nothing after a failed save, though D1 says it never continues on the unsaved restamp.
The Info and Warn rows assert the captured line but not that it appears once, which the acceptance row for the empty-frame log states.
**Fix:** Name how the save failure is forced, assert that the fake starter saw no `Start`, and assert exactly one captured line.

### [LOW] No non-dead terminal prior row for RecoveryRetry

**Location:** `_lyx/plan/24-recovery-count-and-retry.md`, `TestRecoveryRetry` paragraph.
**Issue:** D12 defines the retry for a terminal `dead` recovery-kind prior.
The planned false rows cover no own commits, a fork that committed, no start, a moved HEAD, merge-ins only and two recoveries, but none for a terminal prior that is not dead.
**Fix:** Add a row with a `stuck` or `failed` recovery prior holding commits past its start, expecting false.

### [NIT] RunStdin is the second checked form

**Location:** `_lyx/plan/01-gitexec-run-stdin.md`, Intent.
**Issue:** `gitexec/doc.go` describes two shapes, `Run` checked and `RunGit` raw, so "the third checked form beside `Run` and `RunGit`" contradicts the doc it edits.
**Fix:** Word it as a second checked form beside `Run`, with stdin, and `RunGit` stays the raw form.

### [NIT] fabric.go header is not a target

**Location:** `_lyx/plan/18-fabric-sandboxed-role-guard.md`, Edit list.
**Issue:** The Intent says the package comment at the top of `fabric.go` names the guard, but `fabric.go` is not a target and the header sits outside the `Command` glyph.
**Fix:** Add `internal/fabriccli/fabric.go` to the Edit list as a file target.

## Rejected

- Card 20 and card 22 are too large (generic fork): rejected, each is one mechanical or compile-coupled unit and splitting would break the build between cards.
- Card 21 adds no new test where D9 asks for one over a signal-parsing fake engine (generic fork): rejected, `TestRecoverBatch_SecondCall` drives `TurnEndedAfter` with the signal-parsing engine over real hook lines, so the existing test covers it.
- `FabricReaderVerbs` returning a fresh slice is unpinned (test-gaps fork): rejected, a test of that property is over-testing a trivial implementation detail.
- Card 11's `planparser` import of `internal/pattern` brings `lyxcwd` into the build graph (error-handling fork): rejected, no cycle exists, no test pins the import set, and the card rewords the `doc.go` claims it falsifies.
- Card 16's in-line accept mutates state before a possible failed spawn (error-handling fork): rejected, `recoverBatchCmd` saves state only when it spawned, so nothing persists.
