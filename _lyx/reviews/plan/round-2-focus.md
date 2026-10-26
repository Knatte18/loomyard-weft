---
round: 2
focus:
  - "Affected-set completeness for owner semantics: cards 3 (owners only on root and beam projects, a task reads its beam's owner) and 25 (adopt subject in ways forward) change what a task owner and an adopt command mean. Settle whether any existing test still seeds a task's `owner_machine` directly, or asserts `lyx adopt <task slug>`, without being on the card whose change breaks it; search tests under internal/boardengine, internal/boardcli and internal/battencli, not only those built through shared owner helpers."
  - "Card 25 and the cancel path: `internal/battencli#judgeOrigin` refuses from the report entry and, when origin is unreachable or the worktree is absent, from `cancelDeps.Owner`. Settle whether every branch that reaches `cancelOwnedElsewhere` gets an adopt subject that names a beam or root, never a task slug that `internal/adoptcli#namedEntries` refuses, and whether every function whose signature that requires is on a card."
  - "PATTERN fit of cards 2, 10 and 17: card 2 adds `runproc.EndRole`, and cards 10 and 17 end a beam's dobby through it. Settle whether the `pattern/PATTERN-run-process-owner.md` background and the PATTERN entry's wording still match the code after those cards, and whether a card owns any needed edit."
  - "Beam-Await re-entry on card 15: a pawl run paused or halted after the accepted request is removed and before Done re-enters Beam-Await. Settle whether the card names what the operator does when the beam's dobby is down at that point, and what bounds the re-entry."
---

Focus derives from the round's gating finding and the design-class findings that would change a card's structure.
