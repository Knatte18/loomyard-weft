---
round: 1
ledger:
  - key: card10-comments-fixed-column-wrapped
    rounds: [1]
    status: resolved
  - key: busy-refusal-observation-assertions-vacuous
    rounds: [1]
    status: resolved
  - key: ispending-skip-untested-on-failed-retry
    rounds: [1]
    status: resolved
  - key: detectandfileanomalies-doc-ungrammatical
    rounds: [1]
    status: resolved
---

Round 1 reviewed `5ee34d63e..HEAD`, the generation that rework round 1 started (cards 10, 11 and 12).
It raised four findings, and the fixer resolved all of them in separate commits:

- Card 10's comments used fixed-column wrapping and now use semantic line breaks (`76b24a0ba`).
- The busy-refusal test now seeds a status matching the handoff marker, so its observation assertions can fail, and it proves the bootstrap did not run by the loom seed's absence (`e7a32008a`).
- A new test pins the `isPending` skip for a pending title whose retry fails and that the same pass detects again (`89a890e92`).
- `detectAndFileAnomalies`' doc sentence is reworded (`f34df1aa7`).

The verify commands pass on HEAD, and nothing remains open.
