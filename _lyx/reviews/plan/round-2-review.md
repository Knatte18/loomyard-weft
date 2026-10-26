---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    location: _lyx/plan/03-owners-on-projects.md, _lyx/plan/25-adopt-subject-ways-forward.md
    summary: TestRunOwnerVerdict in internal/loomcli is on no card, yet card 3 breaks one of its rows and card 25 breaks its struct comparison
    fix: |
      In card 3's Edit list add `internal/loomcli_test#TestRunOwnerVerdict` (the test is in package loomcli_test, tier1).
      In card 3's Intent, in the paragraph beginning "Test fixtures that gave a task an owner of its own", add that `TestRunOwnerVerdict`'s `RunHeldEntryWithNoOwnerReadsOwnedHere` row now reads the prime's owner `there`, since the run-held rule goes: the row's expected verdict becomes `{Owner: "there", Elsewhere: true}` and its name says so.
      In card 25's Edit list add `internal/loomcli_test#TestRunOwnerVerdict`.
      In card 25's Intent, in the `loomcli` bullet, add that `TestRunOwnerVerdict`'s expected verdicts carry the adopt subject `root` for the root task, and that `runOwnerVerdict`'s doc comment drops its sentence "so a run-held task with no owner recorded reads as owned here".
      Checked with `lyx loom validate-plan --overlay`: admitted, together with the fixes of F2, F3, F5 and F6.
  - id: F2
    severity: MEDIUM
    class: scope
    location: _lyx/plan/03-owners-on-projects.md
    summary: Card 3 deletes the resume owner claim and the claim's owner write, but two battencli tests asserting both are named only on card 25 and only for wording
    fix: |
      In card 3's Edit list add `internal/battencli#TestRunCmd_ResumeDispositions` and `internal/battencli#TestWire_CreateWorktreeClaimsTheBoardEntry`.
      In card 3's Intent, in the bullet "Batten's resume claim writes no owner", add that `TestRunCmd_ResumeDispositions` drops its four rows that reach `resumeOwnerClaim` (`unowned_run_held_entry_is_recorded_before_the_resume`, `owner_write_with_origin_unreachable_refuses_naming_the_rerun`, `refused_owner_push_refuses_naming_its_way_forward`, `lost_owner_write_refuses_naming_the_other_owner`) and its doc comment drops the run-held owner-write clause.
      In the same card, add that `TestWire_CreateWorktreeClaimsTheBoardEntry`'s `RealSetterRecordsTheOwnerBesideTheClaim` row asserts the Worktree-Create status written and no recorded owner (`wantOwner` empty, the row renamed to say the claim records the status alone), since `Claim` writes no owner.
      Card 25's mention of the same two tests stays, for the adopt wording of their other rows.
      Checked with `lyx loom validate-plan --overlay`: admitted.
  - id: F3
    severity: LOW
    class: scope
    location: _lyx/plan/03-owners-on-projects.md
    summary: TestSetOwners' fixture helpers newOwnerFixture and setOwnerOnOther seed task owners that card 3 makes unreadable, and are not named
    fix: |
      In card 3's Edit list add `internal/boardengine#newOwnerFixture` and `internal/boardengine#setOwnerOnOther`.
      In card 3's Intent, in the `TestSetOwners` test bullet, add that `newOwnerFixture` seeds `away` and `held` as members of a beam project owned by `box` instead of tasks carrying `box` as their own owner, and that `setOwnerOnOther` records its owner on a beam project, so the rows that read `away` lost to `box`, the takeover of `held` and the `alpha` lost race still reach an owner.
      Checked with `lyx loom validate-plan --overlay`: admitted.
  - id: F4
    severity: MEDIUM
    class: decision
    location: _lyx/plan/02-end-processes-by-role.md
    summary: PATTERN.md says runs end only via runproc.End while cards 10 and 17 end the dobby through EndRole, and no declared PATTERN change lets any card reword that line
    fix: |
      No admissible plan edit: the PATTERN.md line and the background's opening sentence must change together, and a changed entry must be declared in the decision record's `## PATTERN changes`.
      Through the parent, add to the decision record a `PATTERN-run-process-owner` change whose line reads "A run's processes: registered in `internal/runproc`, ended only via `runproc`'s `End`, `EndSparing` or `EndRole`.", with the same background link.
      Then the overview declares it, card 2 adds `PATTERN.md` to its Edit list with the whole line in a fenced block, and card 2's background rewrite adds the opening sentence "a run's processes are ended only through `runproc.End`" to the lines it names.
      Checked with `lyx loom validate-plan --overlay`: the overview declaration without the decision-record entry reports `pattern-changes-mismatch` and `pattern-line-unprescribed`, so the plan edit is withheld until the decision record carries it; the fixer defers this finding with that check as the reason.
  - id: F5
    severity: MEDIUM
    class: scope
    location: _lyx/plan/25-adopt-subject-ways-forward.md
    summary: The orch role stencil still tells the orch that lyx adopt of a run's slug is the takeover, a command card 4 refuses for a task, and no card carries the line
    fix: |
      In card 25's Edit list add `contracts/stencils/orch/orch-template-role.md`.
      In card 25's Intent, in its final sentence "The docs name the subject.", add that the role stencil's line "`lyx adopt <slug>` from this prime is the takeover, and the operator decides it" names the run's beam or `root` instead: "`lyx adopt <beam>`, or `lyx adopt root` for a root task, from this prime is the takeover, and the operator decides it".
      Checked with `lyx loom validate-plan --overlay`: admitted.
  - id: F6
    severity: MEDIUM
    class: scope
    location: _lyx/plan/25-adopt-subject-ways-forward.md
    summary: loomcli's localOwnerVerdict, the unreachable-origin branch of runOwnerVerdict, is on no card and has no report entry to take the adopt subject from, so its refusal would name a task slug
    fix: |
      In card 25's Edit list add `internal/loomcli#localOwnerVerdict`.
      In card 25's Intent, in the `loomcli` bullet, replace "the owner verdict carries the subject from the report entry" with "the owner verdict carries the subject from the report entry, and `localOwnerVerdict`, which has no entry, fills it through `Owners.AdoptSubject` over the `parentOf` it already holds".
      Add to the Tests paragraph that `TestRunOwnerVerdict`'s two unreachable-origin rows assert the subject, `root` for the root task.
      Checked with `lyx loom validate-plan --overlay`: admitted.
---

### [MEDIUM] TestRunOwnerVerdict is on no card

**Location:** `_lyx/plan/03-owners-on-projects.md`, `_lyx/plan/25-adopt-subject-ways-forward.md`; the test is `internal/loomcli/ownercheck_integration_test.go`'s `TestRunOwnerVerdict`.
**Issue:** Card 3 removes the run-held rule from `Owners.Effective`, and the test's `RunHeldEntryWithNoOwnerReadsOwnedHere` row records `there` as the prime's owner and expects an owned-here verdict for a run-held task with no owner of its own.
After card 3 that task reads the prime's owner `there`, so the row's verdict is `{Owner: "there", Elsewhere: true}` and the row fails at the batch gate.
Card 25 then adds the subject to `loomcli`'s owner verdict, and the test compares the whole struct (`got != tt.want`) through `export_test.go`'s alias, so every row's expected value needs the subject.
Card 3's ImpactSummary says every fixture that relied on a task's own owner is on the card, and this one is not; the focus asked for tests outside the shared owner helpers, and this is the one that seeds through `boardengine.SetOwners` and a real hub.

### [MEDIUM] Card 3 breaks two battencli tests it does not name

**Location:** `_lyx/plan/03-owners-on-projects.md`; the tests are `internal/battencli/run_test.go`'s `TestRunCmd_ResumeDispositions` and `internal/battencli/wire_seams_test.go`'s `TestWire_CreateWorktreeClaimsTheBoardEntry`, both tier0.
**Issue:** Card 3 deletes `resumeOwnerClaim`, and four rows of `TestRunCmd_ResumeDispositions` exist to reach it: each asserts `wantSets: ["slug:some-slug"]` or one of the write's failures.
Card 3 names `TestPassExitCode` for the same deletion and not this test.
Card 3 also makes `Claim` write the status alone, and the `RealSetterRecordsTheOwnerBesideTheClaim` row of `TestWire_CreateWorktreeClaimsTheBoardEntry` asserts `task.OwnerMachine == machine` after the real setter ran.
Card 25 names both tests, but only for the adopt wording of their other rows, so no card tells the implementer to drop the rows or to change the assertion.
`battencli` is a package card 3 edits, so its tier0 gate runs both tests and fails.

### [LOW] TestSetOwners' fixture helpers are not named

**Location:** `_lyx/plan/03-owners-on-projects.md`; `internal/boardengine/ownerset_integration_test.go`'s `newOwnerFixture` and `setOwnerOnOther`.
**Issue:** `newOwnerFixture` sets `OwnerMachine = "box"` directly on the tasks `away` and `held`, and `migrateEntries` drops that key at the next load.
`setOwnerOnOther("alpha", "box")` does the same on a second clone for the lost-race row.
`TestSetOwners` is on card 3 and its Intent lists new rows, but the two helpers that give its existing rows an owner are not named, though card 3 names comparable helpers elsewhere (`ownedBy`, `fakeOwners.seams`, `stepOwners`).
Low, because `TestSetOwners` is the only test of the file and its edit forces the helpers' change.

### [MEDIUM] PATTERN.md's run-process-owner line stays stale after EndRole

**Location:** `_lyx/plan/02-end-processes-by-role.md`; `PATTERN.md`'s `PATTERN-run-process-owner` line and `pattern/PATTERN-run-process-owner.md`'s opening sentence.
**Issue:** The overview line says "ended only via `runproc.End`" and the background opens "a run's processes are ended only through `runproc.End`".
Card 2 adds `EndRole` and rewrites two other lines of the background, the "`End` alone" bullet and the review line, and names neither the opening sentence nor the overview line, while cards 10 and 17 end a beam's dobby through `runproc.EndRole`.
The decision record's constraint accepts that route ("ended only through `runproc`, by its role"), and the repository already ends records through `EndSparing` from `pairteardown` and `fabriccli`, so no route outside `runproc` appears; the defect is the entry's text, which names `End` alone beside an edited background that names three ends.
This re-raises the item round 1's fixer deferred, and the deferral's reason holds: the overview line cannot be reworded in this plan, because a changed entry must be declared in the decision record's `## PATTERN changes` and `pattern-changes-mismatch` refuses one it lacks, and the opening sentence has to match that line, as the round-1 fixer left it.
The finding stays open until the parent records the change in the decision record; it is a choice the author must make, not a card edit.

### [MEDIUM] The orch role stencil names `lyx adopt <slug>` for a run

**Location:** `_lyx/plan/25-adopt-subject-ways-forward.md`; `contracts/stencils/orch/orch-template-role.md`, the line "`lyx adopt <slug>` from this prime is the takeover, and the operator decides it".
**Issue:** The line sits under a run another machine owns, and the orch reads the slug as the run's task.
After card 4, `lyx adopt <task>` is refused naming the beam or `root`, so the orch would relay a command the verb refuses.
The decision record's scope covers every stencil that names a task owner.
Card 10 edits the file for the start line only, and card 26's `TestRemovedVerbs_NamedNowhere` scans for `lyx adopt --prime`, not for this form.

### [MEDIUM] localOwnerVerdict has no adopt subject

**Location:** `_lyx/plan/25-adopt-subject-ways-forward.md`; `internal/loomcli/ownercheck.go`'s `localOwnerVerdict`.
**Issue:** Card 25 has `loomcli`'s verdict "carry the subject from the report entry" and `refuseOwnedElsewhere` name it.
`runOwnerVerdict` returns `localOwnerVerdict(...)` when origin is unreachable, which reads the local board's effective owner and has no report entry, so the verdict reaches `refuseOwnedElsewhere` with an empty subject.
The refusal then names nothing or the run's own slug, the form `internal/adoptcli#namedEntries` refuses for a task.
The function holds the `parentOf` resolver `Owners.AdoptSubject` needs, so the fix is local.
The sibling branches (`refuseResumeUnreachable`, `cancelDeps.Owner`) name the subject through `Owners.AdoptSubject` in card 25, which this branch should match.

## Rejected

- Focus 2, the cancel path: `internal/battencli#judgeOrigin` reaches `cancelOwnedElsewhere` from two places, the report entry and `cancelDeps.Owner` for an unreachable origin or an absent worktree.
  Card 25 gives the first the entry's `AdoptSubject`, which `pairteardown.OwnerReader` fills through `Owners.AdoptSubject`, and the second a subject read in `hubCancelDeps` through `Owners.AdoptSubject` with `hubgeom.ParentOf`; `cancelOwnedElsewhere`, `cancelDeps`, `hubCancelDeps`, `cancelFake`, `cancelFake.deps` and `TestCancelRun` are all on the card, and no other caller of either exists.
  A beam slug names itself, a task its beam or `root`; not a defect.
  The same question for `loomcli` is F6.
- Focus 4, Beam-Await re-entry on card 15: the card names the dobby-down notice with `lyx beam start <slug>` and `lyx beam land <slug>`, tells a re-entry from a first entry by the await marker, and its status write on entry lifts the hold, so `lyx beam start` passes.
  The notice is the row's only action and a failed `DobbyLive` read or marker write posts nothing, which bounds it; not a defect.
- Focus 3, the other half: card 10's `BeamDown` and card 17's `stopBeamDobby` pass the owner the dobby registers under (the beam's slug, `runproc.HubOwner` for the prime, as `beamcli/dobbydeps.go` registers it), the `RoleDobby` role and `os.Getpid()`, so they match `EndRole`'s contract and the record the dobby writes.
- `TestNewTask`'s subtest that a task keeps `owner_machine`, and `TestUpsertFieldAllowlist`'s `owner_machine_is_accepted_only_as_the_effective_owner`: the first is on card 3 as `TestNewTask` and `normalizeTask` makes its expectation change with the card; the second still passes, since `checkOwnerPayload` keeps refusing a different owner naming `lyx adopt` and the root owner.
