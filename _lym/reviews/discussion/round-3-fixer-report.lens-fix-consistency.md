```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the round-3 fix to the decision record agrees with the sections, criteria and cross-references around it.

Findings:

1. **MEDIUM.** Changed text: D5's new sentences on the kept task tip and on the GitHub merge commit.
   - Summary: D5's opening sentence was left as it was, and the new sentences now contradict it.
   - Evidence: the opening still says "Right after the parent-side merge, and before the parent push, Finalize writes the landing record into the kept directory: the landing commit."
     The fix adds "Before Finalize first merges the parent branch into the task branch, it writes the task's own tip into the landing record".
     It also adds that a pull-request merge "records the merge commit GitHub reports for it, before the parent-side merge".
   - So the record is now written at three points, not only after the merge, and it holds the kept tip as well as the landing commit.
     The opening sentence says it is written at one point and holds one field.

2. **LOW.** Changed text: D5's sentence that the record and row carry no landing commit and no landing time, and say which of the two causes applies.
   - Summary: D5's list of the row's fields was not updated to match.
   - Evidence: the list still reads "The row holds the slug, … the landing commit, the landing time as that commit's committer time, …".
     It names neither field as optional, and it has no field for the reason the row now has to state.
   - D6's changed line ("or the row's reason when it has none") depends on that reason field.

3. **LOW.** Changed text: the new Notes bullet about `pullrequest.Client.Merge`, and D5's GitHub merge-commit sentence.
   - Summary: the sibling note was not updated to name the new write site.
   - Evidence: the note still says "The landing commit is read in `Finalize.recordLanding` … at the merge and again after an archived parent merge".
     D5 now also records a landing commit at the GitHub merge in `mergeOnGitHub`, and the note does not name that site.

The D3 reference ("D1's exclude line"), D1's rationale, the D1 `.tmp-*` criterion, `PATTERN-fabric-boundary`'s single exclude writer and the resume criterion in `D5:` (line 181) all agree with the change.

Further tool calls:
- I grepped `_lym/discussion/support-log.md` for references to D1's `.gitignore` and D5's up-to-date premise, because my context had not shown that file.
  The only hit, line 58, is a historical record of the F4 fix, so it is not stale.
