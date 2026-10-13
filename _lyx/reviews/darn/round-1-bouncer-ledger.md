---
round: 1
ledger:
  - key: pack-in-separate-file-not-last-section
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
---

Round 1 raised one finding (F1): the diff writes the one-turn read instruction and the pack to a separate pack file and ends the prompt with a pointer line, where the board entry wants them inline as the prompt's last section.
The fix changes the rendering contract (`RenderForkPrompt` signature, `BeginBatch` write step, a stencil), so it needs a fresh review.
