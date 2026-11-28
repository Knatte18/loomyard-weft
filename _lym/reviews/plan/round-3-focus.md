---
round: 3
focus:
  - "Card 7 (_lym/plan/07-push-side-core.md) makes coalescePushSides refuse an unset run-records paths seam before the push lock. Which tests, in package fabricengine and outside it, reach that refusal directly or through callers such as CoalescePushRequest and CoalescePushBothAt, does a card give each of them a way to set the seam, and does card 3's (_lym/plan/03-required-seam-installer.md) account of its search match what the search returns?"
  - "Card 7 maps protected-branch refusals to a retry outcome. For every path that can produce such a refusal, including repushOverOriginContained through resetUnnamedTip, updateRemoteBranch and gitrepo.UpdateRemoteBranchLeased as well as a refused push, do the recorded class, the error text and the git processes run match what the decision record, with its added entries, asks for?"
  - "Card 12 (_lym/plan/12-board-test-moves.md) moves board tests into boardengine_test. Does any helper or fixture it plans there hand-write a shed run record, restate support that stays in use in fabricengine_test, add a second notifier fake, or wrap hubforge in a type of its own (PATTERN-test-isolation, PATTERN-test-stand-ins, decision record D9 and D12)?"
  - "Where card 12 or another card relies on a shared testkit, gitkit, boardkit or hubforge helper, new or existing, does that export exist or have a card that creates it, can both consuming test packages import it without a cycle, and are every existing caller of a moved helper covered by a card's targets?"
---

The entries cover the parts of the plan that carry round 2's open defects and the text a fix to them would add.
