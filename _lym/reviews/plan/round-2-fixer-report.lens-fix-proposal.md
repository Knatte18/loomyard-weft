```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals the fixer carried out, through the fix-proposal lens.

**[BLOCKING] F2's proposal leaves a beam's normal landing permanently refused**
- Changed text: card 11, "`pusherDeps` wires the seams for the landing target and for every beam pair target, one holding no landing included", together with card 10's new ceiling paragraph.
- Case: a beam pusher passes between Finalize's parent-side merge and `RecordLanding`. That gap is what F2 describes.
  - With parent seams now on that target, `settleParent` runs, because the parent branch carries an unrecorded landing commit.
  - Step 4 then refuses, since `RunPastFinalize` reads the run "still at Finalize". It writes a `held_back` record through `RecordRefusal`.
  - From then on `TargetRefused` reads the beam target refused, so no tick and no `RecordLanding` wake starts its pusher (card 6: "a refused target starts no pusher on any tick").
  - `SettleRefused` runs only the re-record and steps 1 and 2. Those clear a record only on a settle or a drop, so they never clear a step-4 refusal once the landing is recorded.
- Result: a routine beam landing that meets the gap stays refused until someone pushes by hand. Before the fix, a beam target with no landing ran no parent step, so this refusal could not happen.
- Evidence: card 11, step 4 ("refused the same way only while `RunPastFinalize` reads that run still at Finalize … Each refusal writes one `held_back` record"); card 11, "`SettleRefused` re-records as a pass does, then runs steps 1 and 2 alone"; card 6, the refused-target start rule.
- The gap needs to read as owed rather than refused, or Finalize needs to hold the beam's push lock across the merge and the record.

**[MEDIUM] F7's proposal strands a beam's own commits behind a parent target's mark-only wait**
- Changed text: card 7, "`PushUnset` … No spawn sets the push-now mark", and the overview's spawn decision.
- Case: a beam pair target's pass leaves every landing standing, with none pushed, none held back and none owed. Card 5 then has it wait on its push-now mark between parent passes, not on the interval.
  - A later ordinary commit on the beam spawns `lym loom pusher` without `--push-now`.
  - The spawned process exits at the held lock and sets no mark, so the running pusher never wakes.
  - The beam's own code and records commits wait until a landing, a settle or a teardown happens to set a mark.
- Before the change, every spawn set the mark and woke the pusher.
- Evidence: card 5 ("A parent target whose pass pushed nothing and left every landing standing … waits on its push-now mark between parent passes, not the interval"); card 7's new Bound ("the second process exits at the lock … and the running one takes the commit at its next pass"). In this wait state there is no next pass.

**[LOW] F1's archive tag still sends the held commit to origin**
- Changed text: card 9, "`archiveWeftTip` … pushes the records branch only up to `<hold>^`, while the archive tag still names the whole local tip."
- Case: `tagAndPushArchiveTip` pushes a tag at the local tip, so origin receives the held commit as an object reachable from `archive/<slug>/…`.
  - Origin's records branch lacks it, which is all card 19's new row asserts ("origin's records branch lacks the held commit").
  - Any clone that fetches tags still gets the held records commit, and card 9 never says this exposure is acceptable under D5.
- Evidence: `internal/fabricengine/archive.go:146`, where `tagAndPushArchiveTip(rec, weftRoot, slug, tip, "weft", sleep)` is called with the full `tip`.

I made no further tool calls.
