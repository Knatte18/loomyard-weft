---
round: 1
ledger:
  - key: launch-script-lifecycle-down-replace-failed-add
    rounds: [1]
    status: resolved
  - key: pwsh-executionpolicy-vs-never-fail-launch
    rounds: [1]
    status: resolved
  - key: q10-scrollback-readers-as-acceptance-criterion
    rounds: [1]
    status: resolved
  - key: behaviour-unchanged-by-construction-overclaim
    rounds: [1]
    status: resolved
  - key: decision-record-leanness-rejected-alternatives-restatements
    rounds: [1]
    status: resolved
---

Round 1 was the first round, so there was no prior ledger.
The reviewer raised five findings against the discussion pair for reed's launch-script feature.
The blocking one was a lifecycle rule tied to `RemoveStrand` alone, resting on the false premise that `Down` keeps strands persisted for `Resume`.
The others were an ExecutionPolicy risk that contradicted the never-fail-a-launch rule, a plan-time instruction filed as an open risk (Q10), an overclaim that sourcing a file behaves exactly like a typed line, and leanness leaks into `decision-record.md`.
The writer revised the pair before judging, and all five are resolved in the current text.
The lifecycle is now keyed to the persisted strand table, pwsh dot-sources a script block over `ReadFile`, Q10 is an acceptance criterion, and the rationale is scoped.
The leanness cuts moved deliberation into the support log and kept every requirement in Decisions.
Round 1 closed APPROVED with no open entries.
