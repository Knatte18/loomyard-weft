---
round: 2
focus:
  - "internal/landingshed/finalize_gate_test.go, TestFinalize_GateAndFixer: settle whether the tests pin the approval record, i.e. that an approved fix review writes Reviewed to the attempt record read with ReadFixerAttempts, and that a recorded Reviewed differing from the current HEAD leads to a new fix review rather than a pass (finalize.go judgeFixerChange)."
  - "internal/landingshed/fixreview.go, session end routes: settle whether a test row exists for each of review session timeout, failed session start marked transient, and one re-prompt over an unparseable report followed by an approving report."
  - "internal/landingshed/finalize_gate_test.go, resumed-entry test and over-cap table: settle whether a spent fixer is asserted to spawn no fix review session, and whether the fix review table is asserted to carry the fixer model-spec and a Timeout of FixerTimeoutMin minutes."
  - "internal/landingshed/finalize.go halt for a reviewer that dirtied the tree, plus contracts/specs/llm-model-spec.md and docs/overview.md on the fixer model-spec and fixer_timeout_min keys: settle whether the halt names a single way forward and whether the docs name the fix review session among the sessions those keys govern."
---

The round-1 fixes are new text, so this round reads them as ordinary diff content against the settled scope.
