```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge each proposal the fixer carried out as written (fix-proposal lens).

1. **MEDIUM.** Card 3, the `TestSetOwners` addition: "Its fixture helpers give the existing rows their owner through a beam project … so the rows that read `away` lost to `box`, the takeover of `held` and the `alpha` lost race still reach an owner."
   - **Summary:** Changing the two helpers does not keep these rows working. Every one of them names a task in its owner write, and card 3 makes a task-named write record nothing.
   - **Evidence:** Card 3's own test bullet says "a write naming a task records no owner and reports no `Recorded` slug".
   - `internal/boardengine/ownerset_integration_test.go:173-175` (`TakeoverChangesTheOwnerOfARunHeldEntry`) requests `Slugs: {"held"}` and expects `wantRecorded: {"held"}` and `held: neve`.
   - Line 455 (the lost race) requests `Slugs: {"alpha"}` and then asserts `originOwners(t).Entries["alpha"].Owner == "box"` (line 464). Card 3's `ownersOf` fills `Owner` only for a beam project, so that read gives `""` whatever `setOwnerOnOther` writes.
   - Lines 167-169, 179-181, 492-532 and 542-554 also expect `Recorded`/owner `alpha` for a task-named write.
   - **What the fix needs:** these rows must name the beam project in their request and assertions, for example the beam holding `away`/`held`. The fix's sentence claims they "still reach an owner" through the helpers alone, which is false.

2. **LOW.** Card 3's sentence on `TestRunOwnerVerdict`: "`RunHeldEntryWithNoOwnerReadsOwnedHere` expects `{Owner: "there", Elsewhere: true}` and is renamed".
   - **Summary:** After card 3 the renamed row adds no coverage, and its extra assertion can no longer fail.
   - **Duplicate row:** `ownEntry` no longer has any effect, because a task-named write records nothing. So the row tests the same thing as `OriginRecordingAnotherOwnerReadsOwnedElsewhere`, which also has prime `there` and expects `{there, true}`.
   - **Assertion that can't fail:** the run-held block at `ownercheck_integration_test.go:108-116` checks `owners.Entries[slug].Owner == ""`. That holds automatically once `ownersOf` fills owners only for beams.
   - **Better change:** drop the row, or make it seed something card 3 still distinguishes, such as a run-held member of a beam owned `here` on a prime owned `there`.

3. **LOW.** Card 3's F2 sentence: "`TestRunCmd_ResumeDispositions` drops its four rows that reach `resumeOwnerClaim`".
   - **Summary:** Dropping `unowned_run_held_entry_is_recorded_before_the_resume` outright leaves no row asserting card 3's new behaviour: resuming a run-held entry with no owner writes nothing.
   - **Better change:** keep that row with `wantSets` empty and `wantOutcome: "done"`, renamed to say the resume writes no owner, and drop the other three.
   - **Evidence:** `run_test.go:280-283`, where the row asserts `wantSets: {"slug:"+slug}` today. Card 3's own bullet "Batten's resume claim writes no owner" is the behaviour that row would pin.

I found nothing wrong with the F4, F5 or F6 proposals.

Tool calls:
- Read `ownercheck_integration_test.go:108-125` and the `TestSetOwners` row names and requests in `ownerset_integration_test.go`. My context had only the helpers, not the run-held assertion or the rows that use the helpers.
- Grepped battencli tests for `setErr`/`lost`/`wantSets`, to check whether dropping the four resume rows leaves fake fields unused. It doesn't: `owner_helper_test.go` and `owner_test.go` still use them.
