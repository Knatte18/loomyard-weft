---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:affected-set
    location: _lyx/plan/03-owners-on-projects.md, _lyx/plan/25-adopt-subject-ways-forward.md
    summary: 'Cards 3 and 25 miss four tests that pin a task owner or a slug-named adopt, so each card breaks tests it does not target'
    fix: >-
      Card 3: add these to the Edit list, moving each fixture's task owner onto a beam project or the prime:
      `internal/boardengine#TestFabricReplay` (replay_test.go:330-356 keeps and loses a task owner_machine),
      `internal/boardengine_test#TestRunLock` (the row at runlock_test.go:103 upserts owner box on a held task while the prime owner is neve),
      and `internal/boardcli_test#stepOwners` (verbs_test.go:366-383 asserts that get and list report task owner box).
      Add a sentence to the card's fixture bullet naming the three.
      Card 25: add `internal/battencli#TestRefuseStartOwnedElsewhere` to the Edit list.
      Its rows at owner_test.go:47, 61 and 101 assert lyx adopt plus the slug; make them assert the beam subject (`some-beam`, or the fixture's parent beam `b`).
      Checked with lyx loom validate-plan --overlay together with every other target fix here: no new blocking finding.
  - id: F2
    severity: MEDIUM
    class: scope
    origin: handler
    location: _lyx/plan/25-adopt-subject-ways-forward.md (internal/battencli#judgeOrigin, cancel.go:398-404)
    summary: 'judgeOrigin calls cancelOwnedElsewhere from an owner read with no report entry, so card 25 gives that branch no adopt subject'
    fix: >-
      Card 25: add `internal/battencli#cancelDeps`, `internal/battencli#hubCancelDeps` and `internal/battencli#cancelFake.deps` to the Edit list.
      In the Intent's battencli bullet, say that `cancelDeps.Owner` also returns the slug's adopt subject.
      `hubCancelDeps` reads it through `Owners.AdoptSubject` with the `hubgeom.ParentOf` resolver.
      judgeOrigin's unreachable-origin / no-worktree branch then passes that subject to `cancelOwnedElsewhere`, as its report branch passes the report entry's subject.
      Add a TestCancelRun row: a cancel with origin unreachable, of a task whose beam another machine owns, names lyx adopt and the beam.
      Checked with the overlay (combined run): no new blocking finding.
  - id: F3
    severity: MEDIUM
    class: consistency
    origin: handler
    location: _lyx/plan/02-end-processes-by-role.md
    summary: 'The PATTERN-run-process-owner background still says End alone ends an owner processes, but card 2 adds EndRole and cards 10 and 17 end the dobby through it'
    fix: >-
      Card 2: add `pattern/PATTERN-run-process-owner.md` to the Edit list.
      In the Intent, rewrite its End-only lines ("`End` alone ends the processes an owner holds" and the enforcement line "each end goes through `End`").
      They should name End, EndSparing and EndRole as runproc's ends of an owner's records, EndRole ending one role's records under an owner with End's sequence.
      This matches the decision record's PATTERN-run-process-owner constraint (D4).
      Checked with the overlay (combined run): no new blocking finding.
  - id: F4
    severity: LOW
    class: design
    origin: handler
    location: _lyx/plan/04-adopt-projects.md, _lyx/plan/13-land-blockers.md, _lyx/plan/25-adopt-subject-ways-forward.md
    summary: 'Three Edit members whose parameters or results the Intent changes carry no re-sign arrow'
    fix: >-
      Card 4: write `internal/adoptcli#namedEntries` -> `func namedEntries(deps adoptDeps, owners boardengine.Owners, req request) (slugs []string, root bool, err error)`.
      Naming AdoptSubject needs the deps ParentOf resolver, and the prime result becomes root.
      Card 13: write `internal/battencli#beamMembers` -> `func beamMembers(prime *lyxcwd.Location, slug string) ([]BeamLandMember, error)`.
      Card 25: write `internal/fabricengine#readOriginOwners` -> `func readOriginOwners(l *lyxcwd.Location, seams OriginSeams, targets []originTarget, outcomes []BranchOutcome, slugs []string) (owners, subjects map[string]string, err error)`,
      and `internal/fabricengine#classifyEntries` -> `func classifyEntries(scope ReconcileScope, recorded []string, branches []BranchOutcome, owners, subjects map[string]string) []EntryOutcome`.
      Also add `internal/fabricengine#ReconcileOrigin` to card 25's Edit list, since it calls both (originreconcile.go:733, 758) and caller-uncovered requires it.
      Checked with the overlay (combined run, ReconcileOrigin included): no new blocking finding.
  - id: F5
    severity: LOW
    class: scope
    origin: handler
    location: _lyx/plan/04-adopt-projects.md (internal/fabricengine#refuseDropLocalAnchorChange, anchorstop.go:136)
    summary: 'refuseDropLocalAnchorChange tells the operator to re-run lyx adopt with the moved slug and --drop-local, which namedEntries refuses for a task'
    fix: >-
      Card 4: add `internal/fabricengine#refuseDropLocalAnchorChange` to the Edit list.
      In the Intent's stop bullet, say its way forward ends with re-running the same lyx adopt with --drop-local, never naming the moved slug, since adopt now takes only the beam or root.
      Checked with the overlay (combined run): no new blocking finding.
  - id: F6
    severity: LOW
    class: consistency
    origin: lens:way-forward
    location: _lyx/plan/14-finalize-land-recheck.md (internal/landingshed#landHaltReason)
    summary: 'For a member with a pair or a run in flight, landHaltReason names only the cancel, which leaves the member draft and open on the beam, so the resume halts once more'
    fix: >-
      Card 14: for a member with a pair or a run in flight, landHaltReason names the cancel and then, in the same halt, the lyx board upsert move of its project off the beam, then lyx loom resume.
      This matches card 18's memberHoldRefusal, which already says a draft member still blocks the landing.
      Add the assertion to the matching TestFinalize_LandCheck rows.
  - id: F7
    severity: LOW
    class: design
    origin: lens:way-forward
    location: _lyx/plan/15-pawl-engine.md (internal/pawlengine#BeamAwait.Call, the halted-or-paused bound)
    summary: 'A pawl run paused or halted between the accepted request removal and Done re-enters Beam-Await with the beam dobby stopped, and nothing names lyx beam start'
    fix: >-
      Card 15: when Beam-Await is entered with no request standing, it reads whether the beam's dobby is live through a new Deps seam, DobbyLive.
      Card 17 wires DobbyLive with `dobbyengine.Live` over the beam's `hubgeom.DobbyGeometry`.
      When the dobby is not live, Beam-Await posts the interrupted-landing notice naming lyx beam start <slug> and lyx beam land <slug>.
      Replace the bound bullet with this, and add a TestBeamAwait row: re-entry at Beam-Await with no request and the dobby down posts one notice.
  - id: F8
    severity: LOW
    class: consistency
    origin: lens:way-forward
    location: _lyx/plan/20-batten-cancel-beam.md (internal/battencli#refuseBeamCancel)
    summary: 'A beam cancel with members reuses BeamLandRefusal text, whose land-it option is refused once the pawl run holds the landing hold'
    fix: >-
      Card 20: once the beam's entry reads held (`boardengine.RunHolds`), refuseBeamCancel words each member blocker as card 14's landHaltReason does: the cancel, run in the beam's parent worktree, then the board move.
      It never names lyx batten run <member>.
      While the hold is not held, it keeps BeamLandRefusal's text.
      Add a TestCancelRun row: a beam at running Parent-Align with a member names no lyx batten run.
  - id: F9
    severity: LOW
    class: consistency
    origin: lens:way-forward
    location: _lyx/plan/11-beam-start-stop-status.md (internal/beamcli#startCmd)
    summary: 'A slug given together with --all is refused with no way forward named'
    fix: >-
      Card 11: the refusal names its way forward.
      Drop the slug to start every owned beam, or drop --all to start that one beam.
      TestStart_RefusesBeforeTheReconcile's slug-with-all row asserts both.
  - id: F10
    severity: LOW
    class: consistency
    origin: lens:way-forward
    location: _lyx/plan/01-remove-beam-promote.md (internal/boardengine#checkBeamPayload)
    summary: 'checkBeamPayload way forward names lyx beam create with a slug that today fills with the task own slug, which create refuses'
    fix: >-
      Card 1: word checkBeamPayload's way forward as the card words judgeCreateEntry's.
      Make a project entry with lyx board upsert under a slug of its own, then lyx beam create <that slug>; never the task's slug.
      TestUpsertFieldAllowlist's two rows assert the task's slug is not named after lyx beam create.
  - id: F11
    severity: LOW
    class: consistency
    origin: handler
    location: _lyx/plan/25-adopt-subject-ways-forward.md (internal/battencli#ownedElsewhereRefusal)
    summary: 'After cards 3 and 25, ownedElsewhereRefusal has no production caller left, yet card 25 keeps it'
    fix: >-
      Card 25: add a Delete group with `internal/battencli#ownedElsewhereRefusal`, and drop the Intent clause that keeps it as a slug-subject form.
      Its four callers either move to ownedElsewhereFor or go: owner.go:151 and 201, resumereconcile.go:128, and owner.go:269 in resumeOwnerClaim, which card 3 deletes.
      Its test uses in TestPassExitCode (owner_test.go:183) and TestBattenRun_LostWorktreeCreateClaimExitsWithTheDriversCode (pausehold_test.go:87), both already on card 25, call ownedElsewhereFor.
      Checked with the overlay (combined run): no new blocking finding.
  - id: F12
    severity: LOW
    class: consistency
    origin: handler
    location: _lyx/plan/16-pawl-recipe-rows.md (plan:internal/shedrecipe#TestPawlEntries_RefuseNilSeams)
    summary: 'The new TestPawlEntries_RefuseNilSeams duplicates TestSimpleEntries_UnderfilledEnv, the existing table that blanks each validated Env field and asserts the refusal names it'
    fix: >-
      Card 16: drop the Create of `plan:internal/shedrecipe#TestPawlEntries_RefuseNilSeams`.
      Add `internal/shedrecipe#simpleEntryCases` and `internal/shedrecipe#zeroEnvField` to the Edit list.
      The three pawl entries become rows of simpleEntryCases with validatedFields `Pawl.<Field>`.
      zeroEnvField learns the dotted `Pawl.<Field>` form beside `WebsterDeps.<Field>`, so TestSimpleEntries_UnderfilledEnv and TestSimpleEntries_HappyPath cover them.
      Reword the test paragraph to match.
      Checked with the overlay (combined run): no new blocking finding.
  - id: F13
    severity: LOW
    class: design
    origin: handler
    location: _lyx/plan/18-batten-beam-project.md (internal/battencli#startGuard.refuse)
    summary: 'Card 18 skips the start guard draft check for a beam project, but its let-through line does not name that skip or its bound'
    fix: >-
      Card 18: extend the What-it-lets-through paragraph.
      A draft beam project now starts, bounded by the beam flag only the owner setter's Beam write sets, which lyx beam create makes.
      Dobby still starts tasks only (`dobbyengine` starts.go:37), so only lyx beam create or a hand pass starts it.
      A grouping or unflagged project still refuses at refuseProject.
  - id: F14
    severity: LOW
    class: design
    origin: lens:guard-reach
    location: _lyx/plan/07-dobby-start-named-beams.md (internal/dobbyengine#EnsureLive)
    summary: 'After card 24, no production caller names a slug that is no local beam pair, so the EnsureLive refusal naming lyx beam create is reachable only from its test'
    fix: >-
      Card 7: drop the refusal of a named slug that is no local beam pair, and its TestEnsureLive row.
      Every caller names a worktree it resolved itself: BeamUp its own, --all OwnedWorktrees, Beam-Await its beam, and the anchor move the dobbies it ended.
      Keep the owned-elsewhere and held-beam refusals.
  - id: F15
    severity: NIT
    class: consistency
    origin: lens:way-forward
    location: _lyx/plan/07-dobby-start-named-beams.md (internal/battencli#ensureParentDobby)
    summary: 'The failure way forward names lyx beam start with the parent, with no root form for a run whose parent is the prime'
    fix: >-
      Card 7: say the failure way forward names lyx beam start root when the parent is the prime, as the owner refusal names lyx adopt root.
      TestStartFirstPass asserts it in its prime row.
  - id: F16
    severity: NIT
    class: consistency
    origin: handler
    location: _lyx/plan/16-pawl-recipe-rows.md (internal/runprofile/doc.go:3, 12)
    summary: 'runprofile/doc.go says a profile maps loom or darn and names only their defaults, which goes stale when card 16 adds a pawl block'
    fix: >-
      Card 16: add `internal/runprofile/doc.go` to the Edit list, and say it names the pawl block and the pawl default beside loom's and darn's.
      Checked with the overlay (combined run): no new blocking finding.
  - id: F17
    severity: NIT
    class: consistency
    origin: lens:affected-set
    location: _lyx/plan/09-remove-orch-cleanup.md, _lyx/plan/10-orch-beam-up.md, _lyx/plan/01-remove-beam-promote.md
    summary: 'Comments still name removed verbs after their cards land, and card 26 scans production files only'
    fix: >-
      Card 9: add `internal/cleanupcli#TestPrimeSegments` (its span holds cli_test.go:92, confirmed with lyx quarry resolve) and reword the comment.
      Card 10: add `internal/orchcli#TestCLI_OrchRunnerReadsNoPause` (span 16-46 holds pausegeometry_test.go:17), and say the startdobbies_test.go file header names lyx beam start.
      That file's one member, TestStart_StartsTheDobbiesFirst, is already on card 10.
      A file target there is refused by redundant-file-target and edit-test-file.
      Card 1: say the deletion also drops the ResetRecordsToParent clause from destroy.go's file header (destroy.go:2).
      No target is added: a `internal/fabricengine/destroy.go` target beside card 1's destroy.go members is refused by redundant-file-target, so that part is text only.
      The card 9 and card 10 glyph additions were checked with the overlay (combined run): no new blocking finding.
---

### [MEDIUM] Owner fixtures and adopt assertions missed by cards 3 and 25

**Location:** `_lyx/plan/03-owners-on-projects.md`, `_lyx/plan/25-adopt-subject-ways-forward.md`

**Issue:** Card 3 makes `migrateEntries` and `normalizeTask` drop `owner_machine` from every entry that is not a beam project, and makes a task read its beam's owner or root's.
Three tests still give a task an owner of its own and assert it:
- `TestFabricReplay` (`internal/boardengine/replay_test.go:330-356`) expects a replayed task to keep, or report losing, `owner_machine`.
- `TestRunLock`'s row at `internal/boardengine/runlock_test.go:103` upserts the held task's "own effective owner" `box` under prime owner `neve`. Card 3's `checkOwnerPayload` reads `neve`, so it refuses the upsert.
- `stepOwners` (`internal/boardcli/verbs_test.go:366-383`) expects `get` and `list` to report task owner `box`.

None of the three is on card 3, so card 3's own gate fails.
Card 5 edits `TestRunLock` later, after the break.
Card 25 rewords `refuseStartOwnedElsewhere` to name the adopt subject, but `TestRefuseStartOwnedElsewhere` (`internal/battencli/owner_test.go:47,61,101`) still asserts `"lyx adopt " + slug` and is not on card 25.
The method gap is the same in both cards: the fixture sweep found the tests that build owners through the shared helpers, but not the ones that seed `owner_machine` or the slug-named adopt directly.

### [MEDIUM] judgeOrigin's owner-read branch has no adopt subject

**Location:** `_lyx/plan/25-adopt-subject-ways-forward.md`; `internal/battencli/cancel.go:362-404`

**Issue:** `judgeOrigin` refuses a cancel on a machine that does not own the slug from two places:
- the report entry, when the task worktree is here and origin answered;
- `deps.Owner()`, when origin is unreachable or the worktree is absent (`cancel.go:398-404`).

Card 25 re-signs `cancelOwnedElsewhere(slug, owner, subject)` and takes the subject "from `judgeOrigin`'s report entry".
The second branch has no report, and `cancelDeps.Owner` returns only the machine and the owner.
The implementer must either touch the deps no target lists or pass the slug.
Passing the slug names `lyx adopt <task>`, which card 4's `namedEntries` refuses.

### [MEDIUM] PATTERN-run-process-owner's background is not updated for EndRole

**Location:** `_lyx/plan/02-end-processes-by-role.md`; `pattern/PATTERN-run-process-owner.md`

**Issue:** The background says "`End` alone ends the processes an owner holds" and that review checks "each end goes through `End`".
Card 2 adds `runproc.EndRole`, and cards 10 (`BeamDown`) and 17 (`stopBeamDobby`) end the beam's dobby through it.
The decision record's constraint for this entry permits ending the dobby through `runproc` by its role (D4), and `EndRole` keeps End's signal sequence inside `runproc`, so I read this as background drift, not a breach of the entry.
Even so, the background's rule text and its review line would make a later reviewer flag the new code.
Card 2 extends only `runproc/doc.go`.

### [LOW] Re-sign arrows missing on three Edit members

**Location:** `_lyx/plan/04-adopt-projects.md`, `_lyx/plan/13-land-blockers.md`, `_lyx/plan/25-adopt-subject-ways-forward.md`

**Issue:** Three cards change a member's parameters or results without the re-sign arrow:
- Card 4's `namedEntries(owners, req)` must name `lyx adopt <AdoptSubject>`. That needs a `ParentOf` resolver, which only `adoptDeps` holds, and its `prime` result becomes root.
- Card 13's `beamMembers` changes its result from `[]beamMember` to `[]BeamLandMember`.
- Card 25 carries the subject onto `EntryOutcome.AdoptSubject`. Today `readOriginOwners` returns only an owner map that `classifyEntries` takes (`originreconcile.go:767,813`), so both re-sign. Their caller `ReconcileOrigin` (`originreconcile.go:733,758`) then needs to be on the card, which the gate's `caller-uncovered` names.

### [LOW] An adopt drop-local stop names the moved task

**Location:** `_lyx/plan/04-adopt-projects.md`; `internal/fabricengine/anchorstop.go:120-137`

**Issue:** `refuseDropLocalAnchorChange` refuses a drop-local across an anchor change.
Its way forward names `lyx adopt <slug> --drop-local` again for the slug being dropped.
Under card 4, adopt moves a project's runs, so that slug can be a task, and `namedEntries` refuses a task.
Card 4 rewords `takeOverSlug`'s stop for this reason but not this one, and no other card targets it.

### [LOW] Finalize's member halt takes two halts to progress

**Location:** `_lyx/plan/14-finalize-land-recheck.md`

**Issue:** `landHaltReason` names `lyx batten cancel <slug> --remove`, then `lyx loom resume`, for a member with a pair or a run in flight.
The cancel writes the member back to `draft` (`internal/battencli/cancelcmd.go:461`) and leaves its `project` on the beam.
`beamMembers` still lists it, so the resume halts again, this time naming the board move.
The path progresses only over a second halt.
Card 18's `memberHoldRefusal` already names both steps at once.

### [LOW] An interrupted accept can leave Beam-Await with its dobby down and no notice

**Location:** `_lyx/plan/15-pawl-engine.md`

**Issue:** A passing accept stops the beam's dobby, then removes the request and then the marker before Done.
A pause or halt in that window re-enters through shed's status write as an ordinary wait.
By the card's own bound, that re-entry posts no notice, and with the marker gone nothing restarts the dobby.
The beam then waits with no dobby: no member starts, and card 23's can-land notice cannot fire.
Nothing names `lyx beam start <slug>`.

### [LOW] Beam cancel names a landing the hold refuses

**Location:** `_lyx/plan/20-batten-cancel-beam.md`

**Issue:** `refuseBeamCancel` refuses with `BeamLandRefusal`'s text at every pawl row.
For a member with a pair, that text says to land it with `lyx batten run <member>` (`internal/battencli/beamland.go:79`).
From Parent-Align on, that landing is refused by `ResolveBattenPrime` inside the beam (card 5) and by Worktree-Create's hold read (card 18).
D9 keeps the same wording out of Finalize's halt for this reason.

### [LOW] Slug with --all names no way forward

**Location:** `_lyx/plan/11-beam-start-stop-status.md`

**Issue:** The card has `startCmd` "refuse both together" and adds a refused test row, but it names no way forward.
`PATTERN-refusal-way-forward` asks every refusal to name one.

### [LOW] checkBeamPayload's create way forward can name the task's slug

**Location:** `_lyx/plan/01-remove-beam-promote.md`

**Issue:** Today's text fills its command with the entry's own slug (`internal/boardengine/project.go:82`).
The card's new wording, "`lyx beam create <slug>`", reads the same way, so the message would name a create that refuses the task.
Card 1's other task refusals say "under a slug of its own, then `lyx beam create <that slug>`".

### [LOW] ownedElsewhereRefusal is left with no production caller

**Location:** `_lyx/plan/25-adopt-subject-ways-forward.md`

**Issue:** `ownedElsewhereRefusal` has four production callers today (`owner.go:151,201,269`, `resumereconcile.go:128`).
Card 3 deletes `resumeOwnerClaim`, and card 25 moves the other three to `ownedElsewhereFor`.
Card 25 still keeps the function "for a caller with no board read", but no such caller remains, and its only users are two tests already on the card.

### [LOW] TestPawlEntries_RefuseNilSeams duplicates the under-filled Env table

**Location:** `_lyx/plan/16-pawl-recipe-rows.md`

**Issue:** `TestSimpleEntries_UnderfilledEnv` (`internal/shedrecipe/entries_simple_test.go:240`) already blanks every validated `Env` field of each `simpleEntryCases` row, dotted sub-deps included, and asserts that the refusal names the field.
The planned `TestPawlEntries_RefuseNilSeams` tests the same behavior for three more entries in a new test function.

### [LOW] The draft-check skip carries no stated bound

**Location:** `_lyx/plan/18-batten-beam-project.md`

**Issue:** The card has a beam project skip the start guard's draft check, because a beam cancel leaves its entry `draft` (`cancelcmd.go:461`).
This is a guard bypass, and the "What it lets through" paragraph names neither the skip nor what bounds it.

### [LOW] EnsureLive's no-local-beam-pair refusal is unreachable after card 24

**Location:** `_lyx/plan/07-dobby-start-named-beams.md`

**Issue:** After card 24, every `EnsureLive` caller names a worktree it already resolved:
- `BeamUp` its own (card 10);
- `--all` `OwnedWorktrees` (card 11);
- Beam-Up and Beam-Await their beam (card 17);
- the anchor move the dobbies it ended (card 7).

Card 24 replaces adopt's `StartDobbies`, the one caller that could name an unpaired beam.
The refusal and its `lyx beam create` way forward are then reachable only from `TestEnsureLive`, and guard a state the callers rule out.

### [NIT] ensureParentDobby's failure way forward for the prime

**Location:** `_lyx/plan/07-dobby-start-named-beams.md`

**Issue:** The failure way forward is `lyx beam start <parent>`, but the card gives no `root` form for a prime parent, though its owner refusal names `lyx adopt root`.

### [NIT] runprofile's package doc goes stale

**Location:** `_lyx/plan/16-pawl-recipe-rows.md`; `internal/runprofile/doc.go:3,12`

**Issue:** The doc says a profile maps `loom` or `darn` and names only their defaults.
Card 16 adds a `pawl` block and default but edits only the template and its test.

### [NIT] Comments still naming removed verbs

**Location:** `_lyx/plan/01-remove-beam-promote.md`, `_lyx/plan/09-remove-orch-cleanup.md`, `_lyx/plan/10-orch-beam-up.md`

**Issue:** These comments name verbs or functions the plan removes:
- `internal/orchcli/startdobbies_test.go:3`
- `internal/orchcli/pausegeometry_test.go:17`
- `internal/cleanupcli/cli_test.go:92`
- the `ResetRecordsToParent` clause of `internal/fabricengine/destroy.go:2`

Card 26's scan reads production `.go` files only, so the test comments stay stale silently.
The destroy.go header sits outside every member span card 1 lists.

## Rejected

- guard-reach NIT on `ensureParentDobby` calling its held-parent case a refusal although both callers only warn: the card describes the error the function returns, and its callers' handling of that error is unchanged and already in use for the owner refusal, so the wording is no defect.
- way-forward finding 1 was kept as F6 but at LOW, not MEDIUM: after the cancel, the member has no pair and no run status, so the next halt names the board move and the path progresses, if over two halts.
- way-forward finding 2 was kept as F7 but at LOW, not MEDIUM: the window between the request's removal and Done is a few local writes long.
