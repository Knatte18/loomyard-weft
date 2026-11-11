```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals the fixer carried out as proposed (F2, F3, F4, F5, F7, F8, F9, F10).

1. **MEDIUM — F4, `runWorker` resume branch (`splitSubjects(record.Subjects)`).**
   - A resume now trusts the recorded subjects with no check against the live board.
   - Case: an `--all` adopt stops at owner with subjects `alpha beta`, and `beta` lands or is removed before the resume.
   - Before the fix, `namedEntries` dropped `beta` on resume.
   - Now `beta` stays in `run.Slugs`, so `stepPairs` and `takeOverSlug` can call `AddPair` for it and `stepOwner` can write an owner for it.
   - The new test only covers a beam added since. It does not cover a beam removed since.
   - The only board read before this point is `validateRequest`, which does not resolve `--all` against the recorded set.

2. **LOW — F2, `stepDone` keep/drop built from `run.Req`.**
   - The way forward now applies `--keep-local` or `--drop-local` to every subject of the adopt, not only the held-back projects.
   - On the restart at reconcile, `--drop-local` is also in force for subjects that did not stop. The previous command limited the flag to the held-back set.
   - It is probably harmless, because a subject already adopted is no longer owned elsewhere, so `takeOverSlug` skips its drop. Nothing in the tests pins that.
   - The `stepDone` error text still says "%s held back", while the command beside it names the whole adopt. It reads inconsistently.

3. **LOW — F8, stale sibling in `adoptcli/doc.go` at the line `A reconcile error stops it … the envelope keeps the report and its mutation record`.**
   - The new sentence says the `reconcile` object holds no mutations.
   - The older line still reads as if the report carries them.

4. **LOW — F5, the new text in `pauserecord/doc.go` says the lost exemption and release are deliberate.**
   - The release in `battenshed/innerrun_parentlock.go:36` fires only when `w.pause.OwnerHeld`.
   - A converted adopt entry is covered but not owner-held, so a push child under a parent lock keeps that lock until an operator resumes the slug.
   - The doc states this as intended, but the review left it as a decision. It also does not mention that the lock stays held for the whole pause.

F3, F7, F9 and F10 hold. The F3 test fails if `recordStep` stops clearing `StoppedPart`, because nothing else clears it on a resume at deploy. The F7 test exercises the existing `AlreadyPaused` path with an `AdoptCaller` entry. The `//testtiming:keep` lines match the format of existing markers.

Tool calls: none. My context already held everything this lens needed.
