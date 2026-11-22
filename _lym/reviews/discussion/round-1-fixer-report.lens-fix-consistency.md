```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the fix's change set agrees with the sibling decisions, criteria and cross-references around it (fix-consistency lens).

**LOW: D1's verb list leaves out the steps the fix added.**
- Changed text: D4 steps 2, 3 and 5 of `_lym/discussion/decision-record.md`.
- These steps now use `lym board get`, `git -C <prime> ls-remote origin ...` and `git -C <records repo> fetch --prune origin`.
- D1 still says the move is run "with verbs that exist today" and lists only creel, the tender verbs, cancel, `lym board upsert` and `lym board set-status`.
- The D1 acceptance bullet ("guidance alone with existing verbs") gives the plan writer a list that no longer covers the procedure.
- Fix: add `lym board get` and the two `git -C` reads and fetches to D1's list, or say the list names only the lym verbs that change state.

**LOW: one Rejected alternatives line in the support log now reads against D2.**
- Changed text: D2 now asks the writer "while no pause covers the slug".
- `_lym/discussion/support-log.md` still has "Asking the writer without holding the run: the tip SHA and merge base can drift before the cancel."
- D2 now does ask the writer without a hold, and holds only before the tip is read. A reader can take the old line as rejecting the design D2 adopted.
- Fix: reword it as "Writing the note's commit facts without holding the run", or drop it, since the two new tender-pause lines cover the reasoning.

**LOW: a Turn 1 fact in the support log contradicts the rewritten Constraints.**
- Changed text: the Constraints bullet now says "`archive_tags` names at most the local tip's tag, created but not yet pushed", and D4 step 3 says the code tag is "never read from `archive_tags`".
- Turn 1 in `_lym/discussion/support-log.md` still lists as a settled fact "only `--remove` makes archive tags, and its output lists them as `archive_tags`".
- Turn 2 still records "walkthrough 2 (empty `archive_tags`): fixed, the carrier names no tag." Round 1 drops that rule, and no line points to the reversal.
- Fix: add a note to the Round 1 F2 line that it replaces Turn 2's empty-`archive_tags` rule, or mark the Turn 1 fact as corrected in round 1.

I checked the rest and found it consistent:
- the D4 step numbers referenced from Constraints ("D4 step 3") and Open risks ("D4 step 5");
- D7's pointer to D4 step 3's refusal definition;
- D3's hold now matches D4 step 1;
- the Goal's added parent check;
- the D2, D3, D4 and D7 acceptance bullets;
- D8's glossary line against its acceptance bullet;
- `lym tender resume` as the only lift, in both D4 step 2 and D7;
- the support log's Q6 and Q8 entries against D2 and D4.

Further tool calls: none.
