```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the round-2 changes to the decision record agree with the sections, decisions and criteria around them.

**MEDIUM: D13's two pair-removal conditions now disagree.**
- Changed text: D13 Worktree-Teardown now removes the pair only when "the parent target holds no landing entry of the run (D5)".
- Summary: the sibling sentence for the dobby's landed-pair teardown was left unchanged. It still waits until "its parent target owes no parent push (D5)", and the D13 criterion still says "the parent owes no landing push".
- Evidence: under F2's new D5, a parent target owes a push "while any of its landing entries stands". So the dobby would keep a settled run's pair while another run's landing entry stands, even a refused one, while Worktree-Teardown would remove it. The two removal paths now use different conditions.

**LOW: the new batten-reader sentence in D5 points at the wrong paragraph.**
- Changed text: "reads the run as past its merge only when the parent it judges, chosen as the next paragraph says, holds the run's work."
- Summary: the paragraph after it is now the new owner-read paragraph. The paragraph that chooses the parent is the one that opens "While the parent target owes a parent push, the run reader's past-the-merge read reads the local parent", and it now comes after both inserted paragraphs.
- Evidence: the fix put the owner-read paragraph and the Land-row paragraph straight after the settle text, ahead of that past-the-merge paragraph.

**LOW: three places still use the old single-tip wording.**
- Changed text: D5 now says a landing "records its landing entry", where it used to record "the parent tip as owed".
- Summary: these places were not updated to match:
  - D1 :46: "a landing onto a beam parent records its owed tip and push-now mark on the beam pair's target".
  - D1 criterion :306: "records its owed tip on the beam pair's target".
  - D5 criterion :316: "records the owed parent tip".
- Evidence: D5's new definition makes the record one landing entry per landed run, which names the run, its tip, its held records commit and its pending `done` write.

**LOW: D1 and D8 don't mention the new records hold.**
- Changed text: the hold now binds "the catch-up's `pairteardown.PushRun` (D8), PR-Review's record commit push and cancel's records push", and the catch-up "reads the commits the hold keeps back as owed to the pair's pusher, as D8 reads an owed failure".
- Summary: D1's list of synchronous pushes still says each one "keeps its push". D8 still covers only a push that "fails as owed". Neither says the push stops at the held commit, or how the catch-up treats commits the hold keeps back.
- Evidence: D1 :52-58 and D8 :202-204 are unchanged.

I made no further tool calls; everything I checked was already in my context.
