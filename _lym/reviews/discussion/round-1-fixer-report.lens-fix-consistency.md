```yaml
lens: fix-consistency
outcome: reported
```

fix-consistency lens: checking that the fix's changes agree with the sibling text they touch.

1. **MEDIUM.** D11 step 1, the D11 acceptance criterion and the third Auto-mode assumption disagree with D11's new Bound.
   - Step 1 still says the owner pause comes first "so no pass starts while it archives". The Auto-mode assumption repeats that reason.
   - The D11 criterion says "a failed archive keeps it up, starting no pass".
   - The new Bound says "A dobby whose owner pause failed lets the run's full pass go on at each probe tick as today, while the archive retries".
   - The F19 fix chose the bound option but left the twin claims unqualified. As written, the step and the criterion promise what the Bound denies.

2. **LOW.** D11's Rationale gives a reason the new prefix now works against.
   - It still reads "Tags, not the board's archive branches: `--drop-local` and the teardown already archive there, and the remote-warp cleanup reads them."
   - The next sentence says the `archive-owner/` prefix keeps these tags out of the remote-warp cleanup and every other `archive/` reader.
   - So the first sentence argues for sharing what D11 no longer shares.
   - The support log's twin entry ("Archive branches on origin") already dropped "the cleanup reads them", so the record and the log now disagree.
   - support-log.md:18 says "there is no archive-branch concept", so "archive there" can only mean the `archive/` tags.

3. **LOW.** D10's record match does not cover the beams D4 now covers.
   - D10's new text has `--here` take up a record only when the record "lists the subject in its resolved subject set". D4 now also covers a beam with no owner of its own under an adopted `root` (F2).
   - Take `lym s` in such a beam while a `lym adopt root` record is unfinished and before its owner write. The beam's record is not matched, the Bolt read sees the inherited foreign owner, and `--here` becomes `lym adopt <beam>`.
   - D2 then refuses it as other subjects, naming `lym adopt --discard`. It neither follows nor resumes the root adopt, though D4 treats the beam as part of it.
   - The F2 change reached D4 but not D10's matching rule, and no D10 criterion covers this case.

4. **LOW.** Question ledger Q7 in support-log.md was not updated.
   - It still describes `--here` as "judged after a Bolt pull".
   - D10 now reads the adopt record first and pulls Bolt only when no unfinished record lists the subject.

Tool calls:
- One grep of support-log.md and relevant.md for `archive/`, `owned_here`, the old refusal and record wording, and the resume wording. My context held only lines 60–99 of support-log.md and none of relevant.md, so I checked the untouched lines for stale twins.
