---
round: 4
focus:
  - "Cards 5, 12 and 13, the same-rung retry allowance: settle whether a re-entry attempt, after a batch stop or needs-parent stop on a rung that already holds lost attempts, and a verify re-entry's last-rung fixer, still get one machinery-loss retry, and whether a test row reaches that case."
  - "Card 12, `Run` entry and exit: settle whether the plan removes a prior run's `outcome.yaml` and `summary.md` before a re-step, so `RunInFlight` in card 23 reads a re-stepped run as in flight."
  - "Card 13 `checkStoppedBatch` and card 9 `batchGateSpec`: settle whether the card and batch gate calls come from one rendering with a named source for the last-batch flag, and whether card 13 lists it in Uses."
  - "Card 12 stop paths: settle whether a card names the code that sets `StopRecord` and saves it before `Run` returns Stuck, including a failed-stop Stuck after a last-rung failure, so card 13's `reenter` finds it."
  - "Cards 12 and 23: settle whether the writer and the reader of `outcome.yaml` share one named record type or writer function, or restate the key set in prose."
---

Each entry names a site and a question; none states an expected answer.
