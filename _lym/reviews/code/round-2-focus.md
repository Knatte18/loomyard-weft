---
round: 2
focus:
  - "internal/shedadapters/bouncerverify.go closeVerifiedBlocking and parseVerification: settle whether Go closes a ledger key only when the verification marks it closed and it maps to a finding that fixedBlockingFindings returned, so a key whose finding the fixer disputed, deferred or never answered stays open; check that contracts/stencils/bouncer/bouncer-template-verify.md's output block matches what parseVerification reads."
  - "internal/shedadapters/bouncer_verify_test.go TestBouncer_FixVerification: settle whether the rows pin a partial close over two open BLOCKING keys, a verifier spawn error, a failed ledger write, the whole rewritten ledger, the selection rules of fixedBlockingFindings, and the parseVerification refusals, each asserting a literal value."
  - "internal/shedadapters/bouncerverify.go markLedgerResolved: settle whether a failed write can leave a truncated or lost ledger."
  - "internal/shedadapters/bouncerdecision.go settleDecisionAccept and internal/shedadapters/bouncer.go rejudge: settle whether both archive the round's verification file with the judge outputs and whether rejudge's doc comment matches what the code writes."
---

Focus entries ask questions only and cap no severity.
