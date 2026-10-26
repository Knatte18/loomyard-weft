# Plan round 2 — fixer-report

`lyx loom validate-plan` passes (`ok: true`) after every change below.

## Changes

- **F1** (fixed as proposed, then refined by the self-check): card 3 names `internal/loomcli_test#TestRunOwnerVerdict`, and card 25 names it beside `runOwnerVerdict`.
  Card 3's Intent drops the `RunHeldEntryWithNoOwnerReadsOwnedHere` row instead of renaming it to `{Owner: "there", Elsewhere: true}`, and drops the `runHeld` and `ownEntry` fields with it.
  This is fixed differently: after card 3 the renamed row would duplicate `OriginRecordingAnotherOwnerReadsOwnedElsewhere`, its no-owner-written assertion could no longer fail, and `ownEntry` would record nothing on the root task (card 3: "a task or grouping project named without `Beam` gets no owner").
  Card 25's `loomcli` bullet says the expected verdicts carry the subject `root`.
  It rewrites `runOwnerVerdict`'s doc sentence "It writes no owner, so a run-held task with no owner recorded reads as owned here." to "It writes no owner.", quoting the whole sentence where the proposal quoted a clause.
- **F2** (fixed differently on one row): card 3 names `internal/battencli#TestRunCmd_ResumeDispositions` and `internal/battencli#TestWire_CreateWorktreeClaimsTheBoardEntry`.
  `TestRunCmd_ResumeDispositions` drops the three rows that fail `resumeOwnerClaim`'s write.
  It keeps `unowned_run_held_entry_is_recorded_before_the_resume` with no `wantSets`, renamed to say the resume writes no owner.
  The proposal dropped all four rows; that would leave no row pinning card 3's "Batten's resume claim writes no owner".
  The test's doc comment clause changes to match.
  `RealSetterRecordsTheOwnerBesideTheClaim` asserts the status and an empty `wantOwner`, renamed.
- **F3** (fixed differently): card 3 names `internal/boardengine#newOwnerFixture` and `internal/boardengine#setOwnerOnOther`.
  The proposal moved only the two helpers' seeds onto a beam project.
  That leaves the rows that write an owner naming tasks, which card 3 makes record nothing, and `ownersOf` reads no task owner (`ownerset_integration_test.go:168-181`, `:455-464`, `:492-554`).
  So `TestSetOwners`' bullet now moves each owner write onto a beam project, request and assertions alike:
  - `held`'s and `away`'s rows name the `box`-owned beam project that holds them.
  - `alpha`'s rows and subtests name the beam project `beamer`, and `setOwnerOnOther` records the lost race's owner there.
  - `ClaimStatusLandsInTheSameCommit` becomes the status-only claim on a task.
  - The grouping-project claim row asserts no owner and no status.
- **F4** (decision class, applied on the parent's instruction after it recorded the decision): the overview's `## PATTERN changes` declares `PATTERN-run-process-owner` in the decision record's text.
  It carries no background link: `pattern-changes-mismatch` refused the linked form.
  Card 2 lists `PATTERN.md` with the whole new line in a fenced block, its background rewrite names the opening sentence "a run's processes are ended only through `runproc.End`", and its ImpactSummary names the PATTERN entry.
- **F5** (fixed as proposed): card 25 lists `contracts/stencils/orch/orch-template-role.md` and rewrites its line to "`lyx adopt <beam>`, or `lyx adopt root` for a root task, from this prime is the takeover, and the operator decides it".
- **F6** (fixed as proposed): card 25 lists `internal/loomcli#localOwnerVerdict`, which fills the subject through `Owners.AdoptSubject` over the `parentOf` it holds.
  Its Tests paragraph has `TestRunOwnerVerdict`'s two unreachable-origin rows assert `root`.

## Self-check

- fix-consistency, LOW — `ownEntry` field and comment stale on `TestRunOwnerVerdict`: fixed with the fix-proposal finding below; card 3 drops the field.
- fix-consistency, LOW — card 2's ImpactSummary omits the PATTERN entry: fixed as proposed.
- fix-claims, MEDIUM — the `TestSetOwners` rows name task slugs, so moving the helpers alone keeps nothing working: fixed as proposed; it merges with fix-proposal's identical MEDIUM finding (see F3 above).
- fix-claims, LOW — card 25 quoted a clause as a sentence: fixed as proposed, by quoting the whole sentence and its replacement.
- fix-proposal, MEDIUM — same as fix-claims' MEDIUM: merged, fixed as proposed.
- fix-proposal, LOW — the renamed run-held row duplicates another row and its assertion cannot fail: fixed as proposed, by dropping the row.
- fix-proposal, LOW — dropping all four resume rows leaves the no-owner resume unpinned: fixed as proposed, by keeping one row with no `wantSets`.
- fix-affected-set — no findings; it left F6's search for further `runOwnerVerdict` branches unchecked.

## Disputed

None.

## Deferred

None.
