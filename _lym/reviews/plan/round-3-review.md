---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: scope
    location: _lym/plan/23-retire-owner-pause-callers.md:63
    summary: Card 23 deletes InstallOwnerPause and installedOwnerPause but leaves the unexported ownerPause and ownerPauseMu vars in internal/orchcli/ownerpause.go as dead code
    fix: In card 23's Delete list, after the line for internal/orchcli#installedOwnerPause, add two lines, internal/orchcli#ownerPause and internal/orchcli#ownerPauseMu. In the Intent's bring-up paragraph, after the sentence that names InstallOwnerPause and installedOwnerPause as going, add the clause that ownerpause.go's header comment stops naming the owner pause seam. Do not add the file internal/orchcli/ownerpause.go as a target, since its members are already on the card and redundant-file-target refuses a file listed beside a member. The two Delete additions were checked with lym loom validate-plan --overlay and the run reports no blocking finding the baseline lacks.
---

### [LOW] Dead ownerPause and ownerPauseMu left behind in internal/orchcli/ownerpause.go

**Location:** `_lym/plan/23-retire-owner-pause-callers.md:63`, over `internal/orchcli/ownerpause.go:25`.

**Issue:** `internal/orchcli/ownerpause.go` declares the package vars `ownerPauseMu` and `ownerPause`, and `InstallOwnerPause` and `installedOwnerPause` are their only readers and writers.
Card 23 deletes both functions and the root install in `cmd/lym#newRoot`, and it replaces the `pauseOwned` seam by `endOwned` filled in `wireCLI`.
The two vars and the file's `sync` import then have no user, and the file's header still names "the owner pause seam".
The card does not name the vars, so an implementer following its Delete list leaves them standing.
`code-quality` says to delete what is unused.
`go vet` does not flag an unused package var, so no gate catches it.

## Settled focus questions

These are the verdicts on the round's three focus questions.
None of them is a defect, apart from F1 above, which the third one turned up.

**Card 21's `hand_over` trigger.**
A handed-over run can still list as `hand_over` with `PairPresent` true:
`PairPresent` is `markerExists(pair.WorktreePath())` in `internal/battencli/dobbyreads.go`, a path check,
while `pairteardown.HandOver` removes the pair only when `Teardown.TaskWorktreeGone` reads it as a registered linked worktree, and otherwise archives and returns with the directory standing.
Card 21 answers this with the `handedOver` map: a run that owed the catch-up through this trigger and was then covered by a successful catch-up owes none from it again while it still lists as `hand_over`, and the map forgets it once it stops answering `hand_over` or leaves the list.
A planned `TestRun_CatchUp` row reaches it ("a listed `hand_over` run whose pair stays present after a successful catch-up owes the catch-up once, not on each tick").
Card 20's `DobbyPairSlugs` states the same predicate as `PairPresent` for a pair standing here, so the trigger's set and the catch-up's scope agree, and card 20's `HubCatchUp` reads such a leftover directory as no pair here for fetch, ahead and push, which matches what `HandOver` does with it.
Not a defect.

**Card 15's refusal order.**
`internal/tenderengine/resume.go` `Resume` runs the verdict, then `req.Clear`, then `liftEntries`.
Card 15 places the refusal for a slug only a `bolt-hold` entry covers "after the verdict and before `req.Clear`", so a refused resume leaves the pause intent standing and pushes nothing.
Its `TestResume` rows name "a refused resume of a `bolt-hold`-only slug whose run carries a pause intent leaving `Clear` uncalled and the intent standing", which reaches the refusal and asserts the order.
The verdict does not refuse the resume first, because `resumeAside` gains `Hold` and `runfsm.Aside` already carries a `Hold` field.
Not a defect.

**Card 23's bring-up.**
The card names the replacement seam `endOwned func(location *lymcwd.Location, slug string, endDobby bool) error`, filled in `wireCLI` with `pairteardown.EndSubjectProcesses(prime, slug, endDobby)`.
It reads `endDobby` through an `isBeam func(slug string) (bool, error)` seam filled from `boardengine.ReadOwners(...).Entries[slug].Beam`, which exists on `OwnerEntry`, since `fabricengine.EntryOutcome` carries no beam flag.
It keeps the `owner_pauses` report key, `BeamUpReport.OwnerPauses` and beamcli's `startEntry.OwnerPauses`, and rewords the two warnings to "was not ended" and "could not be ended" with the re-run `lym beam start`.
That matches the parent's Added-after-Discussion decision for card 23, and a planned `TestStart_ReconcilesTheHubFirst` and `startHarness.runWith` assert the calls with the flag, the two warnings and a failed `isBeam` read.
For a task entry, `EndSubjectProcesses(prime, slug, false)` ends the processes registered under that slug through the subject's own pass, so the unset dobby end is right.
The one gap found is F1.
