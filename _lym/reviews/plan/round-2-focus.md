---
round: 2
focus:
  - "Card 9 (`_lym/plan/09-goto-ends-fixers.md`): settle whether a fixer is ended only when a move has passed shed's own goto checks, so a goto refused on a running row, a done run or an unknown target, a target-less relaunch, and a consume that `AdmitRelaunch` refuses each leave the live fixer running; decision D8 licenses ending fixers only when a goto moves the run, and D2's restart path finalizes by waiting on that fixer."
  - "Card 9: if the card now routes the fixer end through a wrapped `GotoGuard`, settle whether every site that hands shedengine a guard (loom's spec wiring, the driver deps' tackler request, the consume attempt's request) carries the wrapper, and whether a test pins both outcomes, ends when the inner guard admits and ends none when it refuses."
  - "Cards 1, 3 and 7: settle whether `internal/runproc/doc.go`, the comments in `internal/websterengine` that name deleted members, and `contracts/specs/shed-recipe-spec.md` are listed under a card's `**Edit:**` and describe the post-change state, with no redundant file or package target beside a member glyph."
---

Round 2 re-checks the card 9 design change in full, since its fix is new text.
The three consistency fixes are checked only for the doc and comment targets they add.
