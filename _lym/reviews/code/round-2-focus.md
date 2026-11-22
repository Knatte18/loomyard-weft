---
round: 2
focus:
  - "Process cap in internal/gatecli/processcap.go: the report steps run beside the last tier with the call's LYM_GATE_TEST_PID env entry. Settle whether processes of a report step can count toward the tier's cap and fail the tier, and whether any new filtering is correct."
  - "Wiring in internal/gatecli/tierrun.go and reports.go: tier test commands are added to and removed from the running-commands set, and the report steps' memory guard spares them through its foreign filter. Settle whether a test pins this production wiring rather than a hand-built set."
  - "lastTierStart in internal/gatecli/run.go and reports.go: settle whether the chain starts beside the first run of the plan's highest tier for multi-tier plans, and whether a test pins that index."
  - "Halt path in runTest: a halted call whose key lists reports must write no report and return only after the chain ended. Settle whether a test pins it and whether the stopped-chain test cancels anything."
  - "Docs that describe when the report steps run: pattern/PATTERN-test-speed.md, internal/gateslot/doc.go, internal/gatecli/doc.go and the GateKey.Reports doc in tiers.go. Settle whether any still says the steps run after the tiers released their slots."
---

Round 2 reviews the whole range again; these entries only name where to look.
