---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: handler
    location: _lym/plan/07-adopt-worker.md (TestJudgeStanding); _lym/plan/08-adopt-caller-follow.md (TestRunsInOrch, TestRunAdopt)
    summary: >-
      Cards 7 and 8 plan tests of unexported helpers and repeated rows whose behaviour other planned tests already pin,
      against PATTERN-test-speed (a test pins behaviour no test covers; an unexported helper is tested through its surface).
    fix: >-
      In card 7, drop the Create glyph `plan:internal/adoptcli#TestJudgeStanding` and its Tests bullet;
      make the TestRunWorker row "Other subjects are refused before any step" assert that the refusal names the recorded subjects,
      the step it stopped at, the re-run and `lym adopt --discard`
      (fresh, resume and follow verdicts are already rows of TestRunWorker and TestRunAdopt, and `--all` matching `--all` is a TestRequestSameSubjects row).
      In card 8, drop the Create glyph `plan:internal/adoptcli#TestRunsInOrch` and its Tests bullet,
      and add its numbered-orch, empty-name and unparsable-name cases as rows of TestRunAdopt's strand rows (spawned args carry or lack `--no-orch`).
      Also in card 8, cut TestRunAdopt's rows "a record that ends stopped exits non-zero", "one that reads interrupted exits non-zero naming the re-run and --discard"
      and the cap half of the last row, which TestFollow already pins, keeping in TestRunAdopt one row that maps a followed end to its exit code.
      The target drops were checked together with every other target fix of this review by `lym loom validate-plan --overlay`, which reported no blocking finding.
  - id: F2
    severity: MEDIUM
    class: design
    origin: handler
    location: _lym/plan/06-adopt-step-order.md (runSteps outcome part); _lym/plan/07-adopt-worker.md (envelopeOf)
    summary: >-
      How a step's outcome part is derived and how envelopeOf merges parts is unspecified;
      the per-slug slugs list, warnings and mutations are written by several steps, so a key-level merge loses fields card 7's own test expects kept.
    fix: >-
      In card 6, define a step's outcome part as the top-level outcome fields that step wrote plus a `slugs` object keyed by slug
      holding only the slugOutcome fields that step set, with the list fields it appended (warnings, mutations, check warnings) holding only its own items.
      In card 7, define envelopeOf as a merge in step order where scalar fields of a later part replace earlier ones,
      `slugs` merges per slug and per field, list fields concatenate, and the slugs list is then rendered in the order `collect` gives.
      Keep card 7's row "a resume after owner still carries the earlier owner step's per-slug owner_recorded" and add to it that the reconcile warnings of the earlier process survive.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:guard-reach
    location: _lym/plan/08-adopt-caller-follow.md (runAdopt follow branch, follow)
    summary: >-
      On the follow verdict follow gets a nil exited that never closes, so a record of another pid is never acted on;
      once the followed worker ends and another adopt rewrites the record within one wake, a plain-shell caller loops forever.
    fix: >-
      In card 8's follow bullets, state that when exited is nil a record naming another pid than workerPID ends the follow:
      it returns an error naming `lym beam status` and the re-run of the same `lym adopt`, saying the followed adopt ended and another started.
      Add a TestFollow row: following with a nil exited, the record turns to another pid's running record, and the follow returns that error at the next wake.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:guard-reach
    location: _lym/plan/07-adopt-worker.md (runWorker step 3, follow verdict); _lym/plan/03-adopt-record.md (WorkerAlive)
    summary: >-
      The worker-side follow verdict is called a fail-loud wiring guard with no reaching test, but WorkerAlive built on proc.IsAlive
      reads an exited, unreaped worker (a zombie, whose ticks still match) as alive, so on-disk state a run produces reaches it.
    fix: >-
      In card 3, make WorkerAlive read `proc.IsRunning` instead of `proc.IsAlive`, so an exited worker that is not yet reaped reads gone.
      In card 7, drop the wiring-guard claim: the follow verdict is a transient refusal naming `lym beam status` and `lym adopt --discard`, written before any step and writing nothing,
      and TestRunWorker gains a row reaching it with a running record whose pid and start ticks are the test process's own.
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:repo-invariants
    location: _lym/plan/08-adopt-caller-follow.md (Command Long); _lym/plan/06-adopt-step-order.md (Command Long)
    summary: >-
      The adopt Command Long is about 1997 bytes against the 2000-byte cap of cmd/lym/clitree_test.go,
      and cards 6 and 8 add the step order, worker, follow, record, --no-orch and --discard to it, which fails the CLI-tree walk.
    fix: >-
      In card 8, replace "its `Long` text gives D3's step order, the worker, the follow and the record" with:
      the Long text names the step order in one line, says a hidden worker runs the steps while the call follows the record,
      names `--no-orch` and `--discard` in one line each, and points to the package doc for the record, resume and envelope,
      staying under the CLI-tree cap; the detail moves to the adoptcli doc.go card 10 rewrites.
      In card 6, make the step-list rewrite of Long the same one-line form.
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:affected-set
    location: _lym/plan/04-green-end-reports.md; _lym/plan/01-adopt-pause-caller.md
    summary: >-
      The cards reword only part of the text set the green end and the hold retirement falsify;
      fixReadError, fixFilingError, battencli OwnedElsewhereError.Error, the planPass comment and the docs/overview.md adopt entry's order and model stay stale.
    fix: >-
      In card 4, add under Edit `internal/adoptcli#fixReadError`, `internal/adoptcli#fixFilingError` and `internal/battencli#OwnedElsewhereError.Error`,
      and add Intent bullets: the two fix-loop texts end with the adopt pauses standing meanwhile in place of the holds,
      and the OwnedElsewhereError text says `lym adopt <subject>` pauses its run, which `lym tender resume <slug>` lets go on.
      Extend card 4's docs/overview.md bullet to rewrite the whole adopt entry: the hidden worker the call follows, the adopt record and its resume,
      the step order with the pause before the owner write and the owner write before the deploy, and a failed deploy leaving the owner written and the runs paused.
      In card 1, add `internal/dobbyengine#planPass` under Edit and reword its comment's hold-caller sentence to the adopt pause, legacy adopt-hold included.
  - id: F7
    severity: MEDIUM
    class: consistency
    origin: lens:affected-set
    location: _lym/plan/06-adopt-step-order.md and _lym/plan/07-adopt-worker.md (TestAdoptIntegration_SecondCloneTakesOver)
    summary: >-
      Both cards list TestAdoptIntegration_SecondCloneTakesOver but neither says that its adopt-back call now returns an error (the record ends stopped)
      or that its --drop-local and --keep-local calls now resume that record.
    fix: >-
      In card 7's TestAdoptIntegration_SecondCloneTakesOver bullet, add: the first clone's adopt back returns an error naming `--keep-local` and `--drop-local`
      with the record stopped at `done`; the `--drop-local` call resumes it from reconcile and ends with the pair-add refusal naming `--keep-local`;
      the `--keep-local` call then records the prime's owner; and the assertion of `PauseLifted` goes.
  - id: F8
    severity: LOW
    class: consistency
    origin: lens:affected-set
    location: _lym/plan/06-adopt-step-order.md (slugOutcome, Delete list)
    summary: >-
      With liftOwnerPauses and the hold targets gone, slugOutcome.PauseLifted, its pause_lifted key, runningHereRun and callerLabel are left with no writer or reader.
    fix: >-
      In card 6, add `internal/adoptcli#runningHereRun` and `internal/adoptcli#callerLabel` under Delete,
      and add to the slugOutcome sentence that `PauseLifted` (`pause_lifted`) goes with liftOwnerPauses.
  - id: F9
    severity: LOW
    class: consistency
    origin: lens:affected-set
    location: _lym/plan/03-adopt-record.md
    summary: >-
      The fabricengine AdoptScratchDir comment and fabricengine doc.go say the directory holds only the adopt check's files,
      which card 3 falsifies by putting adopt.json and adopt.lock there.
    fix: >-
      In card 3, rename the `**Uses:**` header to `**Edit:**`, keep `internal/fabricengine#AdoptScratchDir` under it and add `internal/fabricengine/doc.go`;
      add an Intent sentence that both say the directory also holds the adopt record, its lock and the worker's output;
      and add the line `**ImpactSummary:** one new package read by adoptcli and beamcli; in fabricengine only the AdoptScratchDir comment and one doc sentence change, no code.`
      (placing these edits on card 10 instead is refused by uses-later-target, since card 3 uses AdoptScratchDir).
  - id: F10
    severity: LOW
    class: design
    origin: lens:guard-reach
    location: _lym/plan/06-adopt-step-order.md (stepReconcile); _lym/plan/07-adopt-worker.md (resume)
    summary: >-
      Today's standing anchor-change refusal (refuseStandingMove) is placed nowhere explicitly,
      so a resume past owner reaches the deploy and the check start with no standing-move check.
    fix: >-
      In card 6, state that runSteps runs the standing-move refusal over the subjects and the record's local runs before its first step on every run, resumes included,
      and in card 7 add a TestRunWorker row: a record done through owner, re-run while a move record stands in a subject's worktree, is refused before deploy naming the move's way forward.
  - id: F11
    severity: NIT
    class: consistency
    origin: handler
    location: _lym/plan/02-beam-up-skip-orch.md (Intent, test plan)
    summary: >-
      The card says wantNoOrch asserts no watcher spawn among the calls the fake starter records,
      but the harness counts watcher spawns in h.spawns, not in h.calls.
    fix: >-
      In card 2, reword the wantNoOrch sentence to: with it set, no `start` is among the recorded calls and h.spawns stays zero.
---

### [BLOCKING] Planned tests repeat behaviour other planned tests pin

**Location:** `_lym/plan/07-adopt-worker.md`, `_lym/plan/08-adopt-caller-follow.md`

**Issue:** `TestJudgeStanding` tests the unexported `judgeStanding` directly.
Its verdicts are each rows of the surfaces that call it:
fresh, resume and other subjects in `TestRunWorker`;
follow and other subjects in `TestRunAdopt`;
and `--all` matching in `TestRequestSameSubjects`.
`TestRunsInOrch` tests the unexported `runsInOrch`, whose orch/implementer split `TestRunAdopt`'s strand rows already pin.
`TestRunAdopt`'s stopped, interrupted and cap rows repeat `TestFollow`'s rows.
`PATTERN-test-speed` binds every test to pinning behaviour no test covers, and tests an unexported helper through its surface, so the cards plan redundant tests.

### [MEDIUM] Outcome parts and their merge are unspecified

**Location:** `_lym/plan/06-adopt-step-order.md`, `_lym/plan/07-adopt-worker.md`

**Issue:** Card 6 records each step's outcome part as "the envelope fields the step set, as JSON", and card 7's `envelopeOf` "merges" the parts in step order.
Neither card says how a step knows which fields it set, or how a list merges.
The `slugs` list is written by several steps: pairs, pause, owner and beam.
A resumed process also seeds empty outcomes.
So a key-level merge lets the deploy step's `slugs` replace the owner step's, dropping `owner_recorded`, which card 7's own row expects kept.
`warnings` and `mutations` have the same problem.

### [MEDIUM] Following a live worker never ends once another adopt rewrites the record

**Location:** `_lym/plan/08-adopt-caller-follow.md`

**Issue:** The follow verdict passes a nil `exited`, and the another-pid branch fires only once `exited` closes.
Suppose the followed worker ends and a new adopt writes its record before the next wake, which can be up to eight seconds away.
The follower then never meets its worker's pid again: a plain shell waits forever, and an agent session ends with a misleading `still running`.

### [MEDIUM] The worker's follow verdict is reachable, not a wiring guard

**Location:** `_lym/plan/07-adopt-worker.md`, `_lym/plan/03-adopt-record.md`

**Issue:** `proc.IsAlive` answers true for a zombie (`internal/proc/proc_linux.go:19-24`), and the zombie's start ticks still match the record.
A worker that exits releases `adopt.lock` before its parent reaps it.
In that window, a new worker reads a live worker and returns the follow error.
`refusal-spec.md` exempts only a state no on-disk state a run produces can reach, so the no-reaching-test claim fails.
`proc.IsRunning` is the zombie-aware read on both platforms.

### [MEDIUM] The adopt Long text outgrows the CLI-tree cap

**Location:** `_lym/plan/08-adopt-caller-follow.md`, `_lym/plan/06-adopt-step-order.md`

**Issue:** `internal/adoptcli/cli.go`'s `Long` measures 1997 bytes.
`cmd/lym/clitree_test.go` caps it at `maxLongBytes = 2000`.
Card 8 adds the worker, the follow, the record and two flags to it, and card 6 rewrites its step list, so card 8's gate fails the CLI-tree walk.

### [MEDIUM] Part of the falsified text set is left stale

**Location:** `_lym/plan/04-green-end-reports.md`, `_lym/plan/01-adopt-pause-caller.md`

**Issue:** Card 4 rewords `hubCheckDeps`'s "the holds standing meanwhile", but two texts keep it: `fixReadError` and `fixFilingError` (`internal/adoptcli/fixloop.go:335,340`).
`battencli`'s `OwnedElsewhereError.Error` (`internal/battencli/owner.go:106`) still promises that the adopt check's green end starts the run.
The comment above `planPass` (`internal/dobbyengine/passes.go:108`) still names the hold-caller entry.
`docs/overview.md:782-790` still says the deploy runs before the owner write, that a failed deploy refuses with the holds standing, and that adopt runs in process.
Card 4 drops only two of its sentences.

### [MEDIUM] The integration test's forced change is not described

**Location:** `_lym/plan/06-adopt-step-order.md`, `_lym/plan/07-adopt-worker.md`

**Issue:** After card 6, a held-back project returns an error, so the first clone's adopt back hits the `t.Fatalf` at `internal/adoptcli/adopt_integration_test.go:170`.
The test's later `--drop-local` and `--keep-local` calls now resume the stopped record.
Its `PauseLifted` assertion at line 115 reads a field that is no longer set.
The cards list the test but describe none of this.

### [LOW] Dead outputs and constants remain

**Location:** `_lym/plan/06-adopt-step-order.md`

**Issue:** Once `liftOwnerPauses` and `holdTargets` go, nothing sets `slugOutcome.PauseLifted` (`internal/adoptcli/adopt.go:74`).
`runningHereRun` (`adopt.go:41`) and `callerLabel` (`adopt.go:24`) also lose their last uses, after cards 4 and 6.

### [LOW] fabricengine's scratch-directory docs go stale

**Location:** `_lym/plan/03-adopt-record.md`

**Issue:** `internal/fabricengine/junctionnames.go:188` and `internal/fabricengine/doc.go:1122` describe `AdoptScratchDir` as holding the adopt check's files only.
Card 3 adds `adopt.json` and `adopt.lock` there, and card 8 adds `adopt.out`.

### [LOW] The standing-move refusal has no planned place

**Location:** `_lym/plan/06-adopt-step-order.md`, `_lym/plan/07-adopt-worker.md`

**Issue:** `adopt` refuses a standing anchor change before the reconcile today (`internal/adoptcli/adopt.go:236-244`).
Card 7's `validateRequest` list omits that refusal, and card 6 names it only as "today's reconcile and its refusals".
A resume past `owner` therefore reaches the deploy and the check start with no such check.
D1 names `lym fabric anchor` as the interruption that leaves a record to resume.

### [NIT] Card 2's test sentence names the wrong recorder

**Location:** `_lym/plan/02-beam-up-skip-orch.md`

**Issue:** `startHarness` counts watcher spawns in `h.spawns` (`internal/orchcli/start_test.go:195`), not in `h.calls`.

## Rejected

- guard-reach finding 4: an undecodable record has no `TestRunWorker` or `TestRunAdopt` row. Rejected: the refusal is `adoptrecord.Read`'s own, and `TestRead` reaches it through `Read`. The verbs only propagate it, and `TestDiscard` covers the one verb branch that acts on `ErrDecode`.
- guard-reach finding 1's Windows half: a reused pid on a record without ticks. Rejected as a separate defect: `checkAlive` already accepts reading a record without ticks as alive on the pid alone. F4 keeps the zombie half, which `proc.IsRunning` answers.
- A held-back project held back by an error, such as a refused pair add, ending the record `done`. Not a defect: D3's `done` step ends `stopped` only when a local-commit stop held a project back, and card 6 follows that.
- `followCap` treating a zero keep-warm cap as uncapped. Not a defect: `internal/gatecli/detachfront.go:309` treats `Cap <= 0` the same way.

## Focus notes

- Fidelity: D1–D3 and D5–D9, the Constraints and each acceptance criterion trace to cards 1–10.
  The decision record has no D4 section, so the overview's note that no card plans one is correct.
  D3's held-back stop is carried by card 6's `stepDone`.
  The resume rules (start step, flag replacement, `--discard`) are carried by cards 3, 7 and 8.
- Card order: cards 4, 5 and 9 use only `AdoptCaller` (card 1) and `adoptrecord` (card 3), and no card uses a later card's symbol.
- Attack surface: cards 2, 7 and 8 state what the orch skip, the hidden verb, `--no-orch` and `--discard` let through, and what bounds each.
- Board entry: no card edits this task's board entry or files or promotes a task.
- Edit glyphs and re-sign arrows hold on every card. `BeamUp` keeps its signature, and `adoptDeps.BeamUp` is a struct field.
