---
round: 2
focus:
  - "Where the headroom lint takes its base in the Webster-Burler round gate: `loomshed.NewVerifyGate` in `internal/loomshed/verifygate.go` takes a whole-diff form only when `mergeBase` is non-nil, and only `internal/loomcli/darnwiring.go` sets `env.VerifyMergeBase`; the loom round form diffs from `verifytree.LatestPass` of the plan verify command. Settle whether the decision record names a base that makes branch growth checked in a loom run, and whether D1's premise that every branch's growth ends at or under the line holds under that base."
  - "Behavior after a Publish verify failure: `internal/landingshed/doc.go` keeps the merge-in commit and the way forward is `lyx loom goto --to Webster-Burler`. Settle whether a branch that never touched `PATTERN.md`, with main above the line, passes the round gate under the chosen base, and whether the byte count the finding demands matches what the fixer needs to cut."
  - "Attack surface of the round gate: for rounds with no usable base (no recorded pass, or a recorded pass that is not an ancestor of HEAD) and for a fork that skips its card gate, settle whether a Bound states what growth passes and what limits it."
  - "`PATTERN-refusal-way-forward`: the Webster-Burler verify gate exhausted row in `contracts/specs/refusal-spec.md` lists each failure cause with its way forward. Settle whether a Decision or Constraint requires the new headroom failure to name its way forward, to land its spec row in the same commit, and to be reached by a test."
  - "Scope against the `/proc` acceptance line: `internal/pairteardown/teardown_integration_test.go`, `internal/shedverbs/loop_integration_test.go` and `internal/gatecli/test_integration_test.go` read `/proc/<pid>/stat` directly. Settle whether Scope, D8 and the acceptance line agree on which test files are in or out."
---

Entries restate the sites they depend on; each is a question to settle, not a pre-stated verdict.
