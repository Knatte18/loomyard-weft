---
round: 3
focus:
  - "internal/shedadapters/bouncerdecision.go, escalateUndecided and settleDecisionAccept: when a decision escalation outside the soft-cap window is settled by an arbiter accept on a CONTINUE or CIRCLING round whose ledger holds an open BLOCKING entry of a non-decision class, can acceptRound still pass the round without the parent? Check the fix against the doc comments of routeDecisions and settleDecisionAccept and the TestBouncer_AutoAccept rows at belowBounces and spentBounces."
  - "internal/loomengine/reinterpret.go FlagCodeAutoAccept and the loom Reinterpretation entry in internal/configreg/configreg.go: does a hub that takes the template default of code_auto_accept (no key in loom.yaml), and a hub with no loom.yaml, get the same flag as a hub setting true, while a hub setting false gets none? Check TestFlagCodeAutoAccept's rows, the RunsWithoutFile setting, and the Merge danger section of _lym/landing/summary.md against the rule that a config meaning change is not two-way and its blast radius names every consumer the diff changes."
  - "Hardener focus permission wiring, internal/shedrecipe/entries_burler.go (FocusFindings from harden_round) and internal/burlercli/validate.go checkRoundInRunDirectory: does a test fail when either site is set false or true wrongly, in the hardener and the code run directories?"
  - "internal/shedadapters/bouncerdecision.go escalateUndecided budget escalation detail: does a test assert the detail names the decision finding ids as well as the open BLOCKING keys?"
---

Focus covers the four BLOCKING design findings of round 2, so the fixes get a fresh read.
