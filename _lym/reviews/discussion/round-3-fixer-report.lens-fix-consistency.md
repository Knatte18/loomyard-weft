```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the round-3 change set agrees with the sibling text and criteria it touches (fix-consistency lens).

1. **LOW**: the older D7 criterion (`an attach after the Wait process is killed between steps 1 and 4 ends the run as a strand that died once the bound passes`) and the bound paragraph's closing sentence (`So a Wait process that dies between steps 1 and 4 leaves no run read as relaunching past the bound`) were left as they were. They now overlap with the new criterion.
   - **Summary:** step 4 now contains reed's relaunch. "Between steps 1 and 4" can therefore be read to include a kill after reed's relaunch. The old criterion says that case ends the run as died. The new criterion (`killed between reed's relaunch and the mark's clear … attaches the live run`) and the rewritten bound (`one that reads it over a live strand clears the mark to running and attaches`) say it attaches.
   - **Evidence:** the old criterion predates the fix and sits next to the new criterion in the D7 acceptance criteria.
   - **Suggested wording:** narrow the old criterion to "killed before reed's relaunch in step 4". Change the bound's closing sentence to say "dies anywhere in steps 1 to 4".

2. **LOW**: step 1's rewritten sentence (`every reader of the record reads it as a live running run while its strand is down`) and the later paragraph's rule (`reads a relaunching record within its bound as a live running run`) do not match.
   - **Summary:** step 1 drops the bound qualifier, so the two statements of one rule disagree on whether the live reading expires.
   - **Evidence:** the PATTERN bullet also bounds the reading (`until the mark clears or its bound passes`).

No further tool calls made.
