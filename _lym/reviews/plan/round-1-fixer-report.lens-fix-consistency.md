```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the fix's changes agree with the sibling cards, decisions and cross-references they touch, and find any twin text the fix left unchanged.

Findings:

1. **MEDIUM.** Card 8, `readCancelMarker` bullet (F6 fix), against card 8's `DobbyRuns` and `pairteardown` bullets.
   - **Summary:** the fix closes the window before the intent is written only for readers that call `readCancelMarker`. Its own text still names `DobbyRuns` as covered, but two readers still read the intent alone.
   - **Evidence:** the new text says "without this a start guard, a pre-drive check or `DobbyRuns` would read no cancel in that window". The unchanged bullet still says "`DobbyRuns` sets `RunView.CancelMarker` from the run's status, a cancel intent without a finish, not from a file". `pairteardown`'s `cancelling` is still "read in `New` from the batten record's intent through `ReadBattenRecord`". Neither reader consults the held cancel lock, so `DobbyRuns` and `registeredBattenAlive` still read no cancel in the window. Card 11's `RunView.CancelMarker` doc ("reads an unfinished cancel intent in the record") is the twin and is unchanged too.

2. **LOW.** Card 11, `drivable` and `strandedEnded` bullets (F8 fix), against the unchanged doc texts that define stranded through the pass rule.
   - **Summary:** `strandedEnded` now keeps the paused-with-work-on-parent exclusion even though the dobby gives such a run no pass. Two texts still define a stranded run as one "dobby gives no pass", and no card bullet changes them.
   - **Evidence:**
     - `internal/dobbyengine/ended.go:31-32`: "dobby gives it no pass: batten's state is neither running, nor paused with its work on its parent, nor done".
     - `internal/dobbyengine/doc.go:285` carries the same sentence.
     - Card 11's `strandedEnded` bullet changes only its cancel-marker comment, and the `doc.go` bullet changes only the pass-planning and cancel-marker paragraphs.
   - Once a paused run whose work reached its parent gets no pass, both texts misstate why the run is excluded.

3. **LOW.** Card 4's Intent (F4 fix, "The three reads", the new `FetchedTaskBranchOnOrigin`), against the overview.
   - **Summary:** the overview's fabric-origin-reads Shared Decision and its Card Index line still describe two reads.
   - **Evidence:** `00-overview.md` reads "Card 4 adds two reads to `internal/fabricengine`", and its index line reads "4 — fabric-origin-reads — read origin's fetched parent branch and a child record from origin". Neither names the task-branch read.

4. **LOW.** Card 12's exemption paragraph for `lym loom run` and `lym loom step` (F15 fix).
   - **Summary:** the exemption departs from D1's "Every verb or loop that acts on a run asks the act rule first", but no Shared Decision records it. The overview records the shed-pause aside and the owed-push aside as Shared Decisions, and card 10's goto exemption is likewise card-local.
   - **Evidence:** the decision record's D1 sentence, and the overview's Shared Decisions list, which has no entry for verbs exempt from the act rule.

I made no further tool calls.
