```yaml
lens: fix-claims
outcome: reported
```

Restating the task: check that every claim the fix added or rewrote holds against the code at HEAD.

**MEDIUM: card 3's `TestSetOwners` bullet (F3)**
- **Changed text:** "`setOwnerOnOther` records its owner on a beam project, so the rows that read `away` lost to `box`, the takeover of `held` and the `alpha` lost race still reach an owner."
- **Summary:** moving only the seed onto a beam project doesn't keep those rows working. Their requests still name task slugs, which card 3 makes owner-write no-ops.
- **Evidence:**
  - The rows name tasks:
    - `internal/boardengine/ownerset_integration_test.go:174` requests `Slugs: []string{"held"}, Takeover: true` and wants `wantRecorded: []string{"held"}`, `wantOwners: {"held": "neve"}`.
    - `:180-181` requests `Slugs: {"away","alpha"}` and wants `wantLost: {"away": "box"}`.
    - The lost race at `:455-464` requests `Slugs: {"alpha"}`, wants `Lost == {"alpha": "box"}` and asserts `f.originOwners(t).Entries["alpha"].Owner == "box"`.
  - Card 3's own rules say otherwise:
    - "a task or grouping project named without `Beam` gets no owner".
    - Its `TestSetOwners` row: "a write naming a task records no owner and reports no `Recorded` slug".
    - "`ownersOf` fills `OwnerEntry.Owner` only for a beam project".
  - So once only the seed moves, a write naming `held`, `away` or `alpha` records nothing and loses nothing, and `Entries["alpha"].Owner` reads `""`.
  - For "still reach an owner" to hold, those rows' requests and their owner reads must also move to the beam project or to the effective owner.
  - Other existing rows naming `alpha` as an owner-write target, at `:168-169`, `:202-203`, `:350`, `:492`/`:529-532` and `:542-554`, run into the same rule.

**LOW: card 25's `loomcli` bullet (F1)**
- **Changed text:** "`runOwnerVerdict`'s doc comment drops its sentence "so a run-held task with no owner recorded reads as owned here"".
- **Summary:** the quoted text is a clause, not a sentence.
- **Evidence:** the doc comment at `internal/loomcli/ownercheck.go:43` reads "It writes no owner, so a run-held task with no owner recorded reads as owned here." Calling the fragment a sentence leaves it unclear whether "It writes no owner" stays.

**Claims checked that hold:**
- `TestRunOwnerVerdict`:
  - The run-held row has `recorder: "there"`, prime `there` and no `ownEntry`. Through root that gives `{Owner: "there", Elsewhere: true}`.
  - Its no-owner-written assertion (`:108-116`) still holds.
  - The slug is a root task, so the subject is `root`.
  - The file is in package `loomcli_test`.
- `localOwnerVerdict` takes `parentOf` (`ownercheck.go:72`).
- `TestRunCmd_ResumeDispositions`:
  - Exactly the four named rows carry `wantSets: {"slug:"+slug}` (`run_test.go:283-303`); the other `wantSets` rows are `prime`.
  - The doc comment clause is quoted correctly.
  - `resumeOwnerClaim`'s only production caller is `resumereconcile.go:109`; its only test caller is `TestPassExitCode`'s `LostResumeOwnerClaim` (`owner_test.go:173`), already on card 3.
- `TestWire_CreateWorktreeClaimsTheBoardEntry`'s real-setter row has `wantStatus: "running · Worktree-Create"`. The assertion compares `task.OwnerMachine` to `wantOwner`, so an empty `wantOwner` asserts no owner.
- The stencil line quote matches `orch-template-role.md:144`.
- Card 2's quoted opening sentence matches `pattern/PATTERN-run-process-owner.md:3`. The overview's bullet now matches the decision record's text, and `validate-plan` passes.

**Tool calls:**
1. Read `ownercheck_integration_test.go:108-125`, `resumeOwnerClaim` and its callers: my context lacked the run-held assertions and the claim's callers.
2. Read `owner.go:236-290`, `resumereconcile.go:95-112` and `owner_test.go:150-180`: to confirm what `resumeOwnerClaim` does and who calls it.
3. Grepped `wantSets` in `run_test.go` and the `TestSetOwners` rows naming `away`, `held` or `alpha`: to check the four-row count and F3's claim about those rows.
4. Read `wire_seams_test.go:418-432,484-522`: to check how the real-setter row asserts status and owner.
