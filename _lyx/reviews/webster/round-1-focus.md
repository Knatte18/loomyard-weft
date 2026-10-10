---
round: 1
focus:
  - "Cards 5 and 6: do the guard and the held wait's end read turn-start signals only from the bytes the wait loop's own tick parses, through `SessionSignalParser` by offset, with no read of the shadow fold in `waitshadow.go`, per the shared decision `turn-starts-from-the-loop-read`?"
  - "Card 4: is `PromptOffset` read immediately before the prompt is sent on every send path, with no gap in which the prompt's own turn start can land before the offset?"
  - "Card 3: does the awake clock inject cleanly into both the quiet and the api-error idle clocks, and is the gap arithmetic correct when the awake clock is nil or the machine sleeps across a boundary?"
  - "Card 1: does exempting the Burler's two hand-backs for judgment leave the Bouncer's own check as the one budget on judged rounds, and does a non-judged round still count every hand-back?"
  - "Card 2: do the loom and darn recipe headers and the overview state the PR-Gate episode rule that the engine implements, with no leftover per-task budget wording?"
  - "Tests: does each new case extend the test that already covers its behaviour, per the shared decision `tests-extend-not-add`, and is each added behaviour covered?"
---

Seed focus from the plan's cards and shared decisions; the reviewer judges the diff as code first.
