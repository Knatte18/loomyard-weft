# Card 3 — owners-on-projects

**Create:**
- `internal/boardengine#RootSubject`
- `internal/boardengine#ProjectRecipe`
- `internal/boardengine#Owners.AdoptSubject`

**Edit:**
- `internal/boardengine#Owners.Effective`
- `internal/boardengine#ownersOf`
- `internal/boardengine#OwnerEntry`
- `internal/boardengine#checkOwnerPayload`
- `internal/boardengine#migrateEntries`
- `internal/boardengine#normalizeTask`
- `internal/boardengine#checkProjectFields`
- `internal/boardengine#OwnerRequest`
- `internal/boardengine#writeOwners`
- `internal/boardengine#SetOwners`
- `internal/boardengine#renderReadme`
- `internal/boardengine#renderProjects`
- `internal/boardengine#Store.ListTasksBrief`
- `internal/boardengine#TestOwnersEffective`
- `internal/boardengine#TestSetOwners`
- `internal/boardengine#newOwnerFixture`
- `internal/boardengine#setOwnerOnOther`
- `internal/boardengine_test#TestLoad`
- `internal/boardengine_test#TestNewTask`
- `internal/boardengine_test#TestRenderReadmeGolden`
- `internal/boardengine_test#TestListTasksBriefLayerAndProposal`
- `internal/boardengine#TestFabricReplay`
- `internal/boardengine_test#TestRunLock`
- `internal/boardcli_test#stepOwners`
- `internal/pairteardown#TestTeardown_OriginSeams`
- `internal/battencli#battenCLI.reconcileResume`
- `internal/battencli#TestPassExitCode`
- `internal/battencli#ownedBy`
- `internal/battencli#fakeOwners.seams`
- `internal/battencli#TestArmAt_RunOwnedElsewhereWritesNoSeed`
- `internal/battencli#TestRunCmd_ResumeDispositions`
- `internal/battencli#TestWire_CreateWorktreeClaimsTheBoardEntry`
- `internal/loomcli_test#TestRunOwnerVerdict`
- `internal/tendercli#TestPairOwners`
- `internal/boardcli#storeVerbs`
- `internal/boardengine/doc.go`
- `PATTERN.md`
- `pattern/PATTERN-owner-machine.md`

**Delete:**
- `internal/battencli#battenCLI.resumeOwnerClaim`

**Intent:**
D5: ownership sits on projects.
The root project's owner is the board's `prime_owner`, and a beam project's owner is its `owner_machine`; a task and a grouping project record no owner and read the owner of `hubgeom.ParentOf`'s answer.

- `Owners.Effective(slug, parentOf)`: a slug the board does not hold reads `""`; a beam project (`IsProject && Beam`) reads its recorded owner, else its parent's; every other entry reads its parent's owner; a parent of `""` is the root and reads `o.Prime`.
  The early return for a board with no prime owner goes, so a recorded beam owner holds for the beam and every entry whose `parentOf` answer is that beam, while every entry whose chain reaches the root reads `""` there.
  The run-held rule goes: `RunHeld` no longer makes an unowned entry read `""`, and the field stays only as the fact that a run status stands, which adopt's started-entry test and tender read.
  `ownersOf` fills `OwnerEntry.Owner` only for a beam project, so a stray recorded task owner is never read.
- `Owners.AdoptSubject(slug, parentOf)` names what `lyx adopt` takes to move the slug's ownership: the slug itself for a beam project, else, walking `parentOf`, the nearest beam project, else `RootSubject`; an error from `parentOf` is returned, and a slug the board does not hold is its own subject.
  Every way forward that today names `lyx adopt <task>` names `lyx adopt <AdoptSubject>` from here on; `checkOwnerPayload`'s refusal is the first, and card 25 moves the rest.
- Load drops the retired key, the way `migrate.go` drops `tier` and `type`: `migrateEntries` clears `OwnerMachine` on every entry that is not a beam project, so it leaves `board.json` at the first write after the upgrade, and `normalizeTask` clears it on every entry but a beam project, so an upsert, patch or replay of an older machine's task owner is dropped again.
- `checkProjectFields` accepts `recipe` on a project only when it equals `ProjectRecipe`, and refuses any other recipe naming the field and that a project's recipe is `pawl`; `normalizeTask` refuses `recipe: pawl` on a task naming `loom` or `darn` and that `pawl` belongs to a beam project.
  `ProjectRecipe` mirrors `shedrun.RecipePawl`, which card 6 declares; card 6's `TestBoardRecipe` row pins the two equal, since `boardengine` imports no `shedrun`.
  A beam project with no recipe reads as `pawl` at the seeding site (`battencli.boardRecipe`), which already seeds a project's recipe.
- The owner setter writes only the root owner and beam project owners:
  `writeOwners` writes `OwnerMachine` on a named entry only when it is a beam project after the write; a `Beam` request sets the beam flag and `Recipe = ProjectRecipe` on a named project in the same commit; a task or grouping project named without `Beam` gets no owner.
  A `Claim` request writes the run status alone, never an owner, on a task or a beam project whose effective owner is not another machine, and reports one whose effective owner is another machine in `Lost` unclaimed, so batten's start claim (`claimBoardEntry`) keeps its call and loses its owner write.
  `SetOwners`' re-read judges a claimed slug by its effective owner, `""` and the request's machine both standing as recorded, and an owner-written slug by its recorded owner, as today.
  `OwnerRequest`'s field docs say so.
- Batten's resume claim writes no owner: `resumeOwnerClaim` goes and `reconcileResume` ends with the owner verdict of `resumeRefusal`; `TestPassExitCode` drops its `LostResumeOwnerClaim` row.
  `TestRunCmd_ResumeDispositions` drops the three rows that fail `resumeOwnerClaim`'s write (`owner_write_with_origin_unreachable_refuses_naming_the_rerun`, `refused_owner_push_refuses_naming_its_way_forward`, `lost_owner_write_refuses_naming_the_other_owner`), and keeps `unowned_run_held_entry_is_recorded_before_the_resume` with no `wantSets`, renamed to say a run-held entry with no owner resumes writing no owner; its doc comment's clause that such an entry is recorded as this machine's and each failure of that write refuses becomes that the resume writes no owner.
  Since `Claim` writes no owner, `TestWire_CreateWorktreeClaimsTheBoardEntry`'s `RealSetterRecordsTheOwnerBesideTheClaim` row asserts the Worktree-Create status written and no recorded owner (`wantOwner` empty), renamed to say the claim records the status alone.
- The views: `Store.ListTasksBrief` carries `Effective` for every task and project, never a recorded task owner; `renderReadme`'s Running Owner column shows the effective owner, and `renderProjects` shows a beam's effective owner, both resolving parents through `ParentBeam` over the rendered tasks with the passed prime owner as root's.
- Test fixtures that gave a task an owner of its own now give it through its parent: `ownedBy` puts `owner` on a beam project `some-beam` holding `some-slug`, and `fakeOwners.seams`' `ParentOf` answers `some-beam` for `some-slug` when the owners hold that beam; `TestArmAt_RunOwnedElsewhereWritesNoSeed` records the other machine as the prime owner; `TestPairOwners` moves its task owners onto the prime or a parent beam.
  The tests that seed or assert a task's own `owner_machine` directly move it onto a beam project or the prime: `TestTeardown_OriginSeams`' owner-seam subtest seeds `mine` and `theirs` as beam projects carrying their owners; `TestFabricReplay`'s two owner rows replay the owner change on a beam project entry; `TestRunLock`'s row upserting the held entry's own effective owner splices no `owner_machine` into the held task and records `box` as the prime owner, so `box` is the task's effective owner through root; `stepOwners` seeds the `box` owner on a beam project holding `owned`, so `get` and `list` report `box` for both.
  `TestRunOwnerVerdict`'s run-held row goes, since the run-held rule goes: `RunHeldEntryWithNoOwnerReadsOwnedHere` would read the prime's owner `there`, the case `OriginRecordingAnotherOwnerReadsOwnedElsewhere` covers, and its no-owner-written assertion can no longer fail; the `runHeld` field and its assertion go with it, and the `ownEntry` field and its comment clause go too, since a write naming the root task records no owner and every row reads its owner through the prime.

Tests, each a row of the named test:
- `TestOwnersEffective`: a task reads its beam's owner, and root's when it has no beam; a task's recorded owner is ignored; a run-held task with no owner reads its parent's owner; a beam owner holds on a board with no prime owner for the beam and its members, and a root task there reads `""`; a grouping project reads its parent's owner; `AdoptSubject` gives the beam for a member through a grouping project, `root` for a root task and the slug for a beam.
- `TestSetOwners`: a write naming a task records no owner and reports no `Recorded` slug; a claim on a task writes the status alone and a claim on a task whose beam another machine owns is `Lost` with no status written; a `Beam` write sets the flag, the owner and `recipe: pawl` in one commit.
  Its existing owner writes move from tasks onto beam projects, request and assertions alike, since a write naming a task records nothing and `ownersOf` reads no task owner: `newOwnerFixture` seeds `away` and `held` as members of a beam project owned by `box` instead of tasks carrying `box` as their own owner, and the rows that wrote `held`'s or `away`'s owner (`TakeoverChangesTheOwnerOfARunHeldEntry`, `OwnedElsewhereIsSkippedWithoutTakeover`) name that beam project, its takeover recorded and its loss to `box` reported; the rows and subtests that wrote `alpha`'s owner (`RecordedAndPushed`, `OwnedElsewhereIsSkippedWithoutTakeover`, `CompetingWriteLosesToOrigins`, the `DeferredPush_` subtests, `SkipGitWritesTheOwnerWithNoCommit`) name the unowned beam project `beamer` instead, and `setOwnerOnOther` records its owner on that beam project for the lost race, whose origin read asserts `beamer`'s owner.
  `ClaimStatusLandsInTheSameCommit` becomes the row that a claim on a task writes the status alone, and `ClaimOnAProjectThatIsNotABeamRecordsTheOwnerAndNoStatus` asserts no owner and no status on the grouping project, renamed to say so.
- `TestLoad`: a task and a grouping project carrying `owner_machine` load with none, and a write after the load leaves no `owner_machine` on them while a beam project keeps its own.
- `TestNewTask`: `recipe: pawl` on a project is accepted, another recipe on a project and `pawl` on a task are refused naming the fix.
- `TestRenderReadmeGolden`: a running member of a beam shows the beam's owner in the Running Owner column, and a beam with a recorded owner on a board with no prime owner shows that owner in its project section.
- `TestListTasksBriefLayerAndProposal`: a task carrying a stray recorded owner lists its beam's owner, and a root task the prime owner.

`boardcli.storeVerbs`' list help says `owner_machine` is the effective owner, a beam's or root's for a task; `boardengine/doc.go` states the rules above.
`PATTERN.md`'s entry becomes the declared line, and its background file's per-task owner and run-held paragraphs are rewritten to the project rule:

```markdown
- `PATTERN-owner-machine` — Starting, resuming or landing a run: only where its entry is not owned elsewhere; owners only on root and beam projects, set only by `boardengine`'s owner setter; any other entry reads its `hubgeom.ParentOf` owner; a dobby continue pass is no resume. — [background](pattern/PATTERN-owner-machine.md)
```

**ImpactSummary:** the owner rule's one implementation changes, so every `Effective` reader (batten, tender, dobby, the reconcile's owner seam, beam create) reads project ownership with no call-site change; the removed resume claim had one caller, and every fixture that relied on a task's own owner is on this card.

**Commit:** 3: Owners sit on root and beam projects; a task reads its beam's or root's owner
