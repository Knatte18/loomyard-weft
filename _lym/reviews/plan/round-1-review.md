---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:guard-reach
    location: _lym/plan/04-batten-cancel-export.md (plan:internal/battencli#Cancel), used by _lym/plan/06-front-task-package.md (fronttask.Remove step 1)
    summary: An end-only cancel over an unreachable origin returns nil from Cancel, so fronttask.Remove counts the cancel step done, against D8, and names a hand removal the run lock refuses.
    fix: In card 4's Intent, state that Cancel returns an error when cancelRun's report reads EndOnly, naming that origin was unreachable so the cancel ended the run's processes and stopped, with the way forward lym batten cancel <slug> --remove again once origin answers; add a row to card 4's TestCancelRun assertions (or a TestRemove row in card 6 over a fake Cancel returning that error) pinning that the step fails, the record and entry stay, and RemoveError.HandRemoval names lym batten cancel <slug> --remove first.
  - id: F2
    severity: MEDIUM
    class: consistency
    origin: handler
    location: _lym/plan/11-adopt-holds.md (plan:internal/adoptcli#reconcileWarnings and its TestAdopt rows)
    summary: The D2 warning uses the reconcile's own WayForward when set, which for a diverged code branch names git merge, not lym fabric pull in that worktree as D2 and its acceptance criterion require; a test row pins the departure.
    fix: In card 11's Intent, make every reconcileWarnings line name lym fabric pull in that worktree as the way forward (the reconcile's own WayForward may follow it as detail), then the re-run of the same lym adopt; reword the TestAdopt row about a diverged adopted-beam branch carrying a WayForward to assert that the warning names lym fabric pull.
  - id: F3
    severity: MEDIUM
    class: decision
    origin: handler
    location: _lym/plan/13-check-fix-loop.md (plan:internal/adoptcli#failedEnd, filing step 5)
    summary: The fix task carries label gate, which the board template's vocabulary lacks (labels {}), so on any hub whose board.yaml lacks gate validateLabels refuses every fix filing and the fix loop never files.
    fix: Ask the parent with lym creel post --parent and record the answer with lym loom decision add, then carry it on card 13; the recommended answer is that failedEnd files the gate label only when the hub's board vocabulary knows it (read through boardengine's vocabulary), and TestFailedEnd gains the row of a vocabulary without gate filing the task with label bug alone.
  - id: F4
    severity: LOW
    class: consistency
    origin: lens:affected-set
    location: _lym/plan/11-adopt-holds.md and _lym/plan/12-check-green-end.md
    summary: Texts that say lym adopt resumes or starts runs stay unlisted after cards 11 and 12 move the resumes and first passes to the green check (ownerHaltReason, heldBeamWayForward, ownerWriteError, beamcli/doc.go line 31, battenshed/doc.go line 52).
    fix: Add to card 11's Edit list internal/adoptcli#heldBeamWayForward, internal/adoptcli#ownerWriteError and internal/beamcli/doc.go, and to its Intent that each, and the lost-owner and read-back errors inside adopt, name the subject's adopt check as what resumes the runs; add to card 12's Edit list internal/landingshed#ownerHaltReason and internal/battenshed/doc.go, and internal/landingshed/doc.go under Uses, with Intent lines that the Finalize owner refusal says the adopt's green check resumes the halted run and that battenshed's doc names the adopt check's green end, not lym adopt, as starting a first pass detached.
  - id: F5
    severity: LOW
    class: consistency
    origin: lens:repo-invariants
    location: _lym/plan/05-adopt-check-per-subject.md (internal/gateslot/doc.go:108)
    summary: gateslot's doc gate-key table still names the check as plain lym adopt check in the prime; card 5 updates only pattern/PATTERN-test-speed.md's table.
    fix: Add internal/gateslot/doc.go to card 5's Edit list, and to its Docs paragraph that the gate table's adopt row names lym adopt check <subject>, run in the subject's worktree.
  - id: F6
    severity: LOW
    class: consistency
    origin: lens:repo-invariants
    location: _lym/plan/05-adopt-check-per-subject.md and _lym/plan/11-adopt-holds.md (file headers of internal/adoptcli/check.go, adopt.go, deps.go)
    summary: The file headers of check.go (gate run in the prime), adopt.go (lifts owner pauses, resumes the runs) and deps.go (run and child resume seams) become false, and neither card says it rewrites them.
    fix: Add to card 5's Intent that check.go's file header names the gate run in the subject's worktree, and to card 11's Intent that adopt.go's header names the hold before the owner write and no resume, and deps.go's header drops the run and child resume seams; list no file self glyph, since each card already targets members of those files and redundant-file-target refuses a file beside its member.
  - id: F7
    severity: LOW
    class: design
    origin: lens:guard-reach
    location: _lym/plan/06-front-task-package.md (plan:internal/fronttask#Settle, the OwnedHere false branch)
    summary: A dobby whose beam reads owned elsewhere exits before it ticks, so Settle's drop of records on a machine that lost the subject runs only in a race, and a stale record can run adopt-check a second time if that machine adopts again.
    fix: In card 6's Intent, state that a record a machine keeps for a project it no longer owns is dropped without its action when OwnedHere first reads true again only if the entry is gone or done and the record predates this machine's latest owner write, or else record the one extra adopt check as an accepted bound in the card's Bound line.
  - id: F8
    severity: LOW
    class: design
    origin: lens:guard-reach
    location: _lym/plan/13-check-fix-loop.md (plan:internal/adoptcli#failedEnd, cases 1 and 4)
    summary: An own fix task whose entry holds a run status before its batten run record reads matches neither case 1 nor case 4 and falls through to a filing that File refuses, so the notice misses the task, state and hand step D6 asks for.
    fix: In card 13's case 1, also treat as open an own fix task whose entry holds a run status with no readable batten run record, naming it as starting; add the row to TestFailedEnd.
  - id: F9
    severity: LOW
    class: consistency
    origin: lens:guard-reach
    location: _lym/plan/11-adopt-holds.md (internal/adoptcli#TestAdopt row a live pair is not held)
    summary: Without --keep-local the live-pair row reaches takeOverSlug's stop, not holdTargets' LivePair exclusion, which only --keep-local reaches, so the exclusion stays untested.
    fix: Reword card 11's TestAdopt row to adopt with --keep-local over a live pair under a beam owned elsewhere and assert the run gets no hold and reports running here.
  - id: F10
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/plan/05-adopt-check-per-subject.md (internal/adoptcli#tierNotice, internal/adoptcli#suiteMessage)
    summary: The Intent changes the parameters of tierNotice (it takes the subject) and suiteMessage (one line per check) but their member glyphs carry no re-sign arrow.
    fix: "In card 5's Edit list, replace `internal/adoptcli#tierNotice` with `internal/adoptcli#tierNotice` -> `func tierNotice(subject string, tier CheckTier, logPath string) string` and `internal/adoptcli#suiteMessage` with `internal/adoptcli#suiteMessage` -> `func suiteMessage(checks []suiteStart) string`; state that checkMarkOf reads the subject from CheckRecord.Subject, so its signature stays."
  - id: F11
    severity: NIT
    class: consistency
    origin: handler
    location: _lym/plan/03-board-front-fields.md (plan:internal/boardengine#Board.FileFrontTask)
    summary: FileFrontTask skips checkFrontPayload yet refuses a payload carrying front or on_land as the upsert paths refuse it, without naming what refuses it.
    fix: In card 3's Intent, say FileFrontTask refuses a payload carrying either key before the upsert, with checkFrontPayload's refusal text, and then upserts with checkFrontPayload skipped.
---

### [BLOCKING] An end-only cancel reads as a done removal step

**Location:** `_lym/plan/04-batten-cancel-export.md` (`plan:internal/battencli#Cancel`), consumed by card 6's `Remove` step 1.

**Issue:** `Cancel` returns only `cancelRun`'s error.
With origin unreachable, `judgeOrigin` sets `endOnly` with no error (`internal/battencli/cancel.go:463-483`), and `cancelRun` returns the report with a nil error before `clearRunPhase` runs (`cancel.go:200-203`), so the entry keeps its run status.
`fronttask.Remove` then counts the cancel done, drops the record, and fails at the board removal on `RunLockedError` (`internal/boardengine/runlock.go:150-152`), naming `lym board remove <slug>` alone, which the same run lock refuses.
D8 says a cancel refusal "such as one when origin cannot be read, fails the step", leaving the record and naming the cancel first.
The green end (card 12) and the pre-filing removal (card 13) both go through this path.

### [MEDIUM] The D2 warning names another way forward than `lym fabric pull`

**Location:** `_lym/plan/11-adopt-holds.md`, `reconcileWarnings` and its `TestAdopt` rows.

**Issue:** D2 says the warning names "its branch, its state and `lym fabric pull` in that worktree", and acceptance criterion D2 requires a rewritten or diverged code branch to be "a warning naming `lym fabric pull`".
Card 11 uses the branch outcome's own `WayForward` when set; for a diverged code branch `stopWayForward` returns `codeDivergedWayForward`, a `git merge` and push (`internal/fabricengine/originreconcile.go:328-330`, `failedpush.go:258-266`), and the card's test row asserts the warning names that way forward.
No `Added after Discussion` entry licenses the change.

### [MEDIUM] The fix task's `gate` label is unknown on hubs whose vocabulary lacks it

**Location:** `_lym/plan/13-check-fix-loop.md`, `failedEnd` step 5.

**Issue:** `validateLabels` refuses a label in neither the types nor the labels list of board.yaml (`internal/boardengine/labels.go:18-19`).
The board template's `labels` is empty (`internal/boardengine/template.yaml`); only this hub's `_bolt/_lym/config/board.yaml` adds `gate`.
On another hub every filing is refused at the create, so the loop never files a fix task and each red check only reports the filing error.
D6 fixes the labels, so the plan cannot settle this alone: the choice goes to the parent.

### [LOW] Texts that say adopt resumes the runs stay unlisted

**Location:** cards 11 and 12.

**Issue:** After card 11 adopt resumes nothing and starts no first pass; card 12's green end does both.
Still saying otherwise: `ownerHaltReason`'s refusal "adopt it here with …, which resumes the halted run" (`internal/landingshed/finalize_owner.go:117`), `heldBeamWayForward` (`adopt.go:670`), `ownerWriteError` and the lost-owner and read-back errors (`adopt.go:306,314,564,566`), `internal/beamcli/doc.go:31` and `internal/battenshed/doc.go:52`.
No card lists them.
The proposed target additions pass `lym loom validate-plan --overlay` together with F5's and F10's.

### [LOW] gateslot's gate table keeps the subject-less check verb

**Location:** card 5; `internal/gateslot/doc.go:108`.

**Issue:** The table row reads `adopt  lym adopt check  adopt  all`; card 5 makes the verb `lym adopt check <subject>` in the subject's worktree and updates only `pattern/PATTERN-test-speed.md`.

### [LOW] File headers of check.go, adopt.go and deps.go go stale

**Location:** cards 5 and 11.

**Issue:** `check.go:1` says the gate runs "in-process in the prime"; `adopt.go:1` says adopt "lifts the owner pauses, resumes the runs"; `deps.go:1` lists "the run resume, the child resume".
Cards 5 and 11 change each and say nothing of the headers.
A file self glyph cannot be added beside the cards' member glyphs of those files (`redundant-file-target`), so the fix is Intent text only.

### [LOW] Settle's ownership drop runs only in a race

**Location:** card 6, `Settle`.

**Issue:** A dobby whose beam reads owned elsewhere exits before ticking, so `SettleFronts` never runs on the machine that lost the subject.
Its record stays; if that machine adopts again, `Settle` finds the entry gone (the other machine already ran the action) and runs `adopt-check` again.
The effect is one redundant check, which the card neither prevents nor states as a bound.

### [LOW] An own fix task between claim and status seed falls through to a refused filing

**Location:** card 13, `failedEnd` cases 1 and 4.

**Issue:** Case 1 reads the batten run record; case 4 reads draft or `fixRunHalted`.
An entry already holding a run status whose run record does not read yet matches neither, so `File` refuses on its open-front check and the notice names the filing error, not the task, state and hand step D6 asks for.

### [LOW] The live-pair row reaches the takeover stop, not the hold exclusion

**Location:** card 11, `TestAdopt`.

**Issue:** For a slug read owned elsewhere, `takeOverSlug` stops a live pair without `--keep-local` (`internal/adoptcli/adopt.go:509-532`), so the row passes on the stop and `holdTargets`' `LivePair` exclusion stays untested.

### [LOW] Missing re-sign arrows on card 5

**Location:** card 5, `tierNotice` and `suiteMessage`.

**Issue:** `tierNotice` today takes `(tier CheckTier, logPath string)` and must take the subject to open its line with it; `suiteMessage` takes one `suiteStart` and must render one line per check from `Checks []suiteStart`.
Both glyphs lack the arrow.

### [NIT] FileFrontTask's refusal of the two keys names no check

**Location:** card 3, `Board.FileFrontTask`.

**Issue:** The Intent says it skips `checkFrontPayload` and also refuses a payload carrying either key "as the upsert paths refuse it", without saying what does the refusing.

## Focus verdicts

- Fidelity: D1, D3–D9 and every Constraint are carried by some card; D2's way forward departs (F2), and D8's failed-cancel rule is not reached (F1). No card adds scope the record does not license; the envelope's `suite` to `checks` rename follows from D5's per-subject checks.
- Attack surface: cards 2, 3, 7, 11 and 12 each state what the change lets through and its bound, card 11 through D4's rules and card 12's Bound line.
- Reachability: the refusals of cards 5, 7, 12 and 13 have named inputs and test rows; F1, F8 and F9 are the gaps.
- Card order: no card depends on a later card; cards 11–14 use only what cards 1–10 introduce.
- Board-entry rule: no card edits this task's own board entry or files a task by an agent; card 13's filing is lym's, licensed by D3 and D8.
- Research steps: none carries one; F3 is a fact the plan left unchecked.
- Edit-card glyphs: F10.
- Granularity and test economy: cards 5, 6, 11, 12 and 13 split along reviewable units; `TestPrimeMark` and `TestGreenEnd`/`TestFailedEnd` cover code `TestRunCheck`'s fakes do not reach, so no duplicate.
- PATTERN fit: no card breaks an entry or the two declared entries.
