```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the review's proposals that the fixer carried out (F1–F4) by building concrete cases where they are wrong.

**1. MEDIUM. F2's way forward for a run owned elsewhere sends the operator down the wrong path once the other machine has landed the run.**
- **Changed text:** D5, "for a run owned elsewhere it is `lym adopt` of the run's subject, its beam or its slug, on this machine, then that push, or the hand-over below".
- **Case:**
  1. Run A lands locally and its parent push is owed.
  2. Machine M2 adopts A, re-runs Finalize from origin and lands A on origin's parent under its own commit.
  3. On this machine the pusher reads A's owner before the push. That read comes first, so the refusal is "run owned elsewhere" and never the diverged case.
  4. Following the named way forward, `lym adopt` of A, takes a run back from the machine that already landed it, which may still be mid-Finalize.
  5. Only the push after that adoption reaches the diverged refusal that says "drop the local landing commit".
- **Why it is wrong:** the right way forward here is the drop, with no adoption. The refusal for a run owned elsewhere should first read whether origin's parent already holds the run's work, the read `adopterFirstRepair` makes, and name the drop when it does.
- **The hand-over alternative is also not runnable:** "the catch-up's or Finalize's" hand-over has no trigger for a run that already returned done. Finalize does not run again, and D8's catch-up runs before a run start. So the refusal offers the operator no verb to run.

**2. MEDIUM. F1's diverged drop leaves the run's pair hold and held records commit unstated.**
- **Changed text:** D5, "the run's landing entry goes with no `done` write, since the landing origin holds made its own."
- **Case:** the entry goes, but this sentence neither releases nor keeps the pair hold, and names no removal of the pair.
  - If removing the entry releases the hold as the settle rule does, the pair pusher pushes this machine's held records commit. That commit is an ended entry, written over records the new owner wrote, and origin reads it as a duplicate.
  - If the hold stays, D13's new teardown check no longer applies, since no entry stands. The pair then sits with a held commit that nothing pushes or drops.
- **Proposed correction:** state that the drop discards the held records commit and removes the pair without pushing it, as the hand-over paragraph does.

**3. LOW. F2's "holds back the whole parent push" overreaches its stated reason.**
- **Changed text:** D5, "A run owned elsewhere holds back the whole parent push, the other runs' entries on that parent included, since the parent target pushes only the newest owed tip."
- **Evidence:** D1 allows a push "only to a tip a landing recorded as owed", and that rule does not limit the push to the newest tip. An entry landed before run A has a recorded tip that does not carry A's merge, so it could be pushed and settled. The rule is a defensible choice, but the reason given is not what forces it, and older runs' `done` writes stall with no need.

No further tool calls made.
