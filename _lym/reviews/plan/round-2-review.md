---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lym/plan/06-adopt-step-order.md (stepReconcile and stepOwner paragraphs)
    summary: "The reconcile warnings are assigned to stepReconcile, but reconcileWarnings needs the adopted beams (known only after the owner write) and each slug's Stopped mark (set by the pairs step), so the unchanged TestAdopt warning rows cannot pass as written"
    fix: "In 06-adopt-step-order.md take 'the reconcile warnings' out of the stepReconcile paragraph and put them in stepOwner: after the read-back it calls reconcileWarnings over the reconcile report, Inputs.OwnerStood as the adopted beams, and the run's merged per-slug outcomes (which carry the pairs step's Stopped marks), and the warnings join the owner step's part so they still precede the deploy warning. Add one sentence that the steps up to owner share one process, which is why the report and the earlier outcomes are in memory there. The TestAdopt rows DivergedBranchOfAnAdoptedBeamCarriesItsWayForwardInParentheses and RewrittenPrimeBranchWithNoWayForwardIsAWarningNamingFabricPull stay unchanged, and card 7's row that the earlier process's reconcile warnings survive a resume stays."
  - id: F2
    severity: MEDIUM
    class: decision
    location: _lym/plan/06-adopt-step-order.md (runSteps, 'A step's error ends the record stopped') and _lym/plan/07-adopt-worker.md (envelopeOf)
    summary: "No rule says whether a step that fails contributes its part to the end envelope; D2 builds the envelope from done steps only, so a refused reconcile or deploy loses the reconcile report, mutation record and deploy outcome that today's refusals carry"
    fix: "Decide it in cards 6 and 7. Preferred: when a step fails, runSteps stores the part the step gathered before failing in the record (a new Record field such as StoppedPart json.RawMessage, cleared when that step later succeeds), and envelopeOf merges it after the done steps without listing the step as done. In card 7 add to the refused-reconcile outcome an assertion that the envelope keeps the reconcile report and the mutations with partial true when the faked reconcile returns mutations and an error, and have card 6's failed-deploy row keep the current assertion that the deploy outcome is named. If the author accepts the loss instead, say so in card 6's Intent and drop the Deploy assertion of the rewritten AFailedDeployRefusesBeforeTheOwnerWrite row."
  - id: F3
    severity: LOW
    class: consistency
    location: _lym/plan/06-adopt-step-order.md (runSteps seeding sentence)
    summary: "runSteps seeds an outcome for every slug in the record's Subjects, which includes root, so an adopt of root gains a root item in the envelope's slugs list that today's envelope never carries"
    fix: "In 06-adopt-step-order.md change the seeding sentence to seed from the record's Subjects without the word root, plus Inputs.Runs and Inputs.Paused, and say that envelopeOf in 07-adopt-worker.md names the Subjects without root to collect, so the slugs list keeps today's members."
  - id: F4
    severity: MEDIUM
    class: consistency
    location: _lym/plan/07-adopt-worker.md (TestRunWorker rows) against _lym/plan/06-adopt-step-order.md (TestAdopt rows)
    summary: "Three TestRunWorker rows (held-back project ends stopped at done, failed deploy ends stopped at deploy, reconcile error ends stopped at reconcile) repeat behavior TestAdopt already pins through the same fake world and runWorker"
    fix: "In 07-adopt-worker.md drop those three rows from TestRunWorker. In the same card's TestAdopt edit, add to the three TestAdopt rows that already cover them (card 6's held-back row, card 6's failed-deploy row, and the existing ReconcileErrorRefuses row in adopt_test.go) one assertion each on the persisted record: state stopped, the step it stopped at, and for the held-back row a way forward naming --keep-local and --drop-local through Request.Command. Reword 'one row per D2 and D3 criterion' to 'one row per D2 criterion the TestAdopt rows do not reach'."
  - id: F5
    severity: LOW
    class: consistency
    location: _lym/plan/01-adopt-pause-caller.md (TestDobbyRuns_PairAtItsOwnAnchor row), _lym/plan/08-adopt-caller-follow.md (TestRunAdopt), _lym/plan/09-beam-status-adopt.md (TestStatus_PrintsThePrimesDobbyAndEachLocalBeams)
    summary: "Three more planned assertions repeat a test that already pins the behavior: the dobby-view AdoptCaller row repeats TestCoverageOf, the interrupted message in TestRunAdopt repeats TestFollow, and the stopped-section wording in the tier1 status test repeats TestAdoptSectionOf"
    fix: "(a) In 01-adopt-pause-caller.md drop the Edit target internal/beamcli#TestDobbyRuns_PairAtItsOwnAnchor and its Tests bullet; battencli copies coverage.Held for any caller and the existing HoldCallerEntryReadsHeldAndOwnerEntryDoesNot row already pins the copy (checked with validate-plan --overlay: no new blocking finding). (b) In 08-adopt-caller-follow.md let the TestRunAdopt rows for stopped and interrupted assert only the exit code and that the envelope or error reaches the writer, since TestFollow pins the interrupted text naming the re-run and lym adopt --discard. (c) In 09-beam-status-adopt.md let the tier1 status test assert only that the adopt section appears with the record's state and step, leaving the way-forward wording to TestAdoptSectionOf."
  - id: F6
    severity: LOW
    class: scope
    location: _lym/plan/06-adopt-step-order.md (Intent), internal/adoptcli/adopt.go lines 1-3, 36-37, 837-840, 899 and 936, internal/adoptcli/deps.go lines 1-4
    summary: "Comments that still state the old behavior (a green check lifts holds, starts runs and resumes the landing) are reworded by no card: the adopt.go file header, the resumeSlug and runBeamSteps and heldBeamWayForward doc comments, and the deps.go file header"
    fix: "Add one sentence to card 6's Intent naming these comments as reworded with the edits of their symbols, with no new targets (a file listed beside its members is refused by redundant-file-target): the adopt.go header, resumeSlug's doc (it lifts nothing and starts nothing), runBeamSteps' doc and heldBeamWayForward's doc (the landing resumes by hand with lym tender resume and lym loom resume), and the deps.go header (the adopt pause in place of the pause lift and hold, and only the tries file)."
  - id: F7
    severity: LOW
    class: consistency
    location: _lym/plan/03-adopt-record.md (Edit list and Intent), internal/lymdirs/dirs.go:40
    summary: "Card 3 rewords the AdoptScratchDir comment and the fabricengine doc to say the directory also holds the adopt record, but lymdirs.AdoptDirName, the declarer of that directory, keeps saying it holds the adopt check's record, output and lock"
    fix: "In 03-adopt-record.md add the Edit target internal/lymdirs#AdoptDirName and a Uses section listing internal/lymdirs/doc.go (the doc-go-unlisted check requires it), and add to the Intent that AdoptDirName's comment names the adopt record, its lock and the worker's output beside the check files. Checked with validate-plan --overlay together with F5(a): no new blocking finding."
  - id: F8
    severity: LOW
    class: consistency
    location: _lym/plan/00-overview.md (Shared Decisions, last bullet)
    summary: "The Shared Decisions say the re-run text of every refusal and way forward is Request.Command, but the cards keep the TestAdopt rows that assert the existing reentry text 'then re-run lym adopt with the same arguments'"
    fix: "In 00-overview.md reword the last Shared Decisions bullet to say Request.Command is the re-run text of every refusal and way forward that the record, the worker, the follow and the stop add, and that the existing reentry text of the takeover's own refusals stays."
---

### [MEDIUM] Reconcile warnings belong to a step that cannot compute them

**Location:** `_lym/plan/06-adopt-step-order.md`, `stepReconcile` (step 1) and `stepOwner` (step 4).

**Issue:** Step 1 lists "the reconcile warnings" among `stepReconcile`'s work.
Today's `reconcileWarnings(report, adopted, outcomes, req)` skips a non-prime branch unless its slug is in `adopted`, the beams whose owner write stood, and skips a slug whose outcome has `Stopped` set.
`adopted` exists only after `stepOwner`, and `Stopped` is set by `stepPairs`.
At `stepReconcile` neither is known, so an adopted beam's unmoved-branch warning would be dropped.
The existing row `DivergedBranchOfAnAdoptedBeamCarriesItsWayForwardInParentheses` (`adopt_test.go:683`) expects exactly that warning, and card 6 and card 7 keep the TestAdopt rows unchanged.
The merge rule in card 7 ("list fields concatenate", step order) also fixes the order, so the owner step is the earliest step that has the inputs and still puts these warnings before the deploy warning, as today.

### [MEDIUM] A failing step's part has no stated fate

**Location:** `_lym/plan/06-adopt-step-order.md` (`runSteps`, "A step's error ends the record `stopped` at that step") and `_lym/plan/07-adopt-worker.md` (`envelopeOf`).

**Issue:** The cards record a part only "after each" step, and `envelopeOf` merges "every step the record lists as done" (D2's wording).
A step that fails is not done, so nothing in the plan carries what it gathered before it failed.
Today every refusal returns the outcome built so far: the reconcile refusal carries the report and the mutation record (`result.Reconcile = report` and `rec.Extend(report.Mutations)` run before the error return, and `adopt.go`'s doc says "the envelope still carries the reconcile and its mutation record"), and the failed deploy carries `result.Deploy` (`adopt_test.go:937` asserts `got.Deploy` is named).
Under the plan, a reconcile that mutated a remote branch and then failed reports no mutations and `partial` false, and a failed deploy no longer names the deploy.
The plan states no rule for it, and card 6's rewritten failed-deploy row does not say whether the deploy stays named.
This is a choice for the author, hence class `decision`: keep the failed step's part (the fix above extends D2 rather than contradicting it, since every done step still contributes) or accept the loss and say so.

### [LOW] Seeding from Subjects adds a root item to the slugs list

**Location:** `_lym/plan/06-adopt-step-order.md`, `runSteps`.

**Issue:** Card 3 defines `Record.Subjects` as "the beam slugs, `root` when root goes", and `runSteps` seeds an outcome for each slug `Subjects` names.
The old `outcomes` map never held `root`; `collect(slugs, outcomes)` listed beam slugs and touched entries only.
A seeded `root` outcome would appear as `{slug: root}` in `slugs` for every adopt of root.
`envelopeOf` also renders the slugs "in the order `collect` gives", and `collect` dereferences `*outcomes[slug]` for every named slug, so naming `root` there would panic without the seed.

### [MEDIUM] TestRunWorker repeats TestAdopt rows

**Location:** `_lym/plan/07-adopt-worker.md`, `TestRunWorker` rows "A held-back project ends the record `stopped` at `done`", "A failed deploy ends it `stopped` at `deploy`" and "A refused step (the reconcile error) ends the record `stopped` at `reconcile`".

**Issue:** After card 7, `TestAdopt` drives `runWorker` through `world.workerDeps`, the same entry and fake world `TestRunWorker` uses.
Card 6's `TestAdopt` rows already cover the held-back project ending with the other subject reaching `suite:` and the failed deploy stopping at `deploy` with the owner recorded, and `ReconcileErrorRefuses` (`adopt_test.go:393`) already covers the reconcile error.
The only new fact in the `TestRunWorker` rows is the persisted record state, which is an assertion on those rows, not a new test.
The testing skill's rule is one test per behavior, with a repeated case as a row or an assertion of the test that already covers it.

### [LOW] Three more repeated assertions

**Location:** `_lym/plan/01-adopt-pause-caller.md`, `_lym/plan/08-adopt-caller-follow.md`, `_lym/plan/09-beam-status-adopt.md`.

**Issue:** (a) `battencli/dobbyreads.go:168` copies `coverage.Held` into the dobby's view for any caller, the existing `HoldCallerEntryReadsHeldAndOwnerEntryDoesNot` row pins the copy, and card 1's `TestCoverageOf` row pins that `AdoptCaller` sets the flag; the added beamcli row tests neither anything new.
(b) `TestFollow` pins that `interrupted` returns an error naming the re-run and `lym adopt --discard`; `TestRunAdopt` repeats that text where it should pin only the exit-code mapping.
This does not re-raise round 1's proposal to cut the mapping rows: the fixer was right that `follow` returns a record or an error and never a code, so the mapping row stays; only the repeated message wording goes.
(c) The tier1 status test asserts the stopped section naming `deploy` and `lym adopt --discard`, the stopped row of `TestAdoptSectionOf`.
The tier1 test is worth keeping for the wiring, not for the wording.

### [LOW] Comments the green end and hold retirement falsify and no card rewords

**Location:** `_lym/plan/06-adopt-step-order.md`; `internal/adoptcli/adopt.go:1-3` ("Each subject's adopt check ... lifts its holds and starts its runs once it comes back green"), `:36-37`, `:837-840` (`resumeSlug`: "the adopt check of its subject starts a held run at its green end", "has its owner pause lifted"), `:899` (`runBeamSteps`), `:936` (`heldBeamWayForward`); `internal/adoptcli/deps.go:1-4` ("the pause lift and hold", "the holds and tries files").

**Issue:** Cards 4 and 6 list the texts they reword and are thorough about error texts, `Long`, and the docs, but these comments sit on symbols the cards edit without naming the comment.
The run-constants comment at `:36-37` is the one card 6 names; the others are left to the implementer to notice.
The `resumeSlug` and `runBeamSteps` comments would be false the moment card 6 lands.

### [LOW] `lymdirs.AdoptDirName` keeps the old description

**Location:** `_lym/plan/03-adopt-record.md`; `internal/lymdirs/dirs.go:40`.

**Issue:** Card 3 rewords `fabricengine.AdoptScratchDir`'s comment and the fabricengine `doc.go` sentence to say the directory also holds the adopt record, its lock and the worker's output.
`lymdirs.AdoptDirName` names the same directory and says it "holds the adopt check's record, output and lock".
`PATTERN-lym-files` makes `lymdirs` the one declarer of that name, so its comment is the first place a reader looks.

### [LOW] Shared Decisions overstate the re-run text

**Location:** `_lym/plan/00-overview.md`, Shared Decisions, last bullet.

**Issue:** "The shared re-run text of every refusal and way forward is `adoptrecord.Request.Command`" would send an implementer through `reentry` (`adopt.go:27`, used by dozens of takeover texts) while rows such as `adopt_test.go:393`, `:643`, `:679` and `:686` assert "then re-run lym adopt with the same arguments" and cards 6 and 7 keep them unchanged.

## Rejected

- Focus 4, the `Long` byte cap: not a defect.
  The current `Long` is about 1,985 bytes of which the step list is about 1,226; card 6 replaces the list with one line and card 8's added lines (worker, `--no-orch`, `--discard`, pointer) leave it well under `maxLongBytes` (2000).
  The new hidden `worker` command fits the walk's form rules (`Use` token `[<subject>...]`, hidden with the internal audience, lower-case `Short`).
- Focus 3, the caller's follow of a hidden worker: not a defect.
  Each reachable record state has a named input and a `TestFollow` row: a record naming `workerPID` that reads `done`, `stopped` or `interrupted` (a `running` record whose pid is gone, which `proc.IsRunning` reads even for an exited worker not yet reaped); `exited` closed with another pid's record started later (matching subjects follow, other subjects refuse); started earlier or absent (one more read, then the error naming `adopt.out`); a nil `exited` with another pid's or no record; the cap and the zero cap.
  The follow ends in every branch once the worker exits, and in the follow verdict it ends on the record's own state.
  A finished worker whose `done` is overwritten by another adopt before the next wake reads as the "another started" branch and exits non-zero, which is the safe direction.
- Focus 1, `TestAdoptSectionOf`, `TestWorkerArgs` and `TestDiscard` test unexported helpers directly.
  Each pins something the surface test cannot reach cheaply (liveness states, the parse-back between the caller's args and the worker's flags, the lock), so they are not flagged.
- The claim in card 1 that `TestPause` and `TestResume` already pin `already_paused` and the lift for a caller other than the operator's holds (`pause_test.go:777`).
- Card 4's claims that no test asserts the old `ownerHaltReason` and `OwnedElsewhereError.Error` texts: a grep of the test files for the old phrases finds none.
- The `PATTERN` entries: no card breaks one.
  The spawn logs through `logger.Info`, resolves the binary through `selfexec.LymBinary`, is reaped by its starter and registers under `runproc.HubOwner`; the record is never-tracked under `fabricengine.AdoptScratchDir`, written through `internal/state`; the follow takes its agent cap from `shuttleengines.KeepWarm` and joins `unboundedWaitFiles`.
- Fixes F5(a) and F7 were checked together with `lym loom validate-plan --overlay`: the baseline `{}` run is ok, and the combined run reports no blocking finding.
  F7's first form, with `internal/lymdirs#AdoptDirName` alone, raised `doc-go-unlisted`, so the proposal lists `internal/lymdirs/doc.go` under Uses.
