---
round: 1
focus:
  - "internal/shedadapters/bouncer.go: with BlockingEscalates set, does a spent budget with an open BLOCKING ledger entry, or an unreadable ledger, always escalate with cause budget and never reach auto-accept or the soft-cap accept? Check every path through settleUndecided and openBlockingKeys."
  - "internal/shedadapters/bouncer.go and internal/burlerengine/roundcheck.go: does a held decision-class finding gate in every settle path (budget, auto-accept, APPROVED verdict) until the parent rules, and does the round check admit a focus-origin finding outside the changed files without admitting unrelated ones?"
  - "contracts/recipes/*.yaml and internal/loomrecipe/testdata: do the Code-Bouncer, Code-Burler-Fix and Harden-Bouncer rows in every recipe set the switches the commit message names, and does any recipe that should carry them lack them or carry them wrongly?"
  - "contracts/stencils/burler/burler-review-format.md, harden/harden-rubric-review.md, bouncer/bouncer-template-escalation.md: do the stencils make a hardener review raise each undriven or WARN focus entry as a BLOCKING decision-class finding with origin focus, and route it to the arbiter and then the parent, without dropping an existing rule?"
  - "Pattern: does the diff cover the decision record's third item where it names the arbiter's scope (internal/shedarbiter, arbiterstop)? Compare against the record's Where list for anything asked but not changed, or changed but not asked."
  - "internal/shedadapters/bouncer_escalation_test.go: the Loosened-Check trailer names TestBouncer_SoftCapRule; does the trailer's cause hold against the diff, and do the test and its doc comment still check what they state?"
  - "internal/loomrecipe/decisionswitches_test.go: the redundancy report lists TestDecisionSwitches_SetOnTheRowsThatAskTheParent as contained, covered by TestAmendmentTarget; compare their assertions and settle the test-economy finding."
  - "Commit message claims: check the Root-Cause trailer against the defects the diff removes, and the summary's claims, if present, against the diff."
---

Seed focus drawn from the decision record's three faults and the diff's file list.
The two uncovered changed blocks in the mutation report are worth checking against the reach report.
