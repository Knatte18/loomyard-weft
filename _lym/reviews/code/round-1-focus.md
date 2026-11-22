---
round: 1
focus:
  - "internal/battencli/commitstatus.go newCommitStatusSeam, the startCore branch: does the condition (marker running row before Run-Shed, st == running, isStartRow) push for every resumed, halted or marker-less start row and skip the push only for an uninterrupted start? Check Worktree-Create, which `isStartRow` excludes, and the Run-Shed boundary."
  - "Scope against the decision record: only the start rows from Seed-Child to Run-Shed stop pushing. Does any later transition, Land and Worktree-Teardown included, lose its push, and does the first later push still carry the unpushed start commits?"
  - "Takeover: the decision record asks that a takeover of a run whose start rows were not pushed redoes those rows and does not break. Does the diff keep that path intact, and does any test or code in the diff cover the takeover meeting the starting machine's next push?"
  - "Mutation survivor at commitstatus.go:225 (negate condition in newCommitStatusSeam), possible killer internal/shedcli.TestParity_Scenario: read the line and the covering tests, and judge whether a check on the diff's own behavior is missing."
  - "startCore carries no Admit, Owed or Sleep deps and a no-op Push: does skipping the gate and the owed-push retry change behavior when a push is owed from an earlier transition?"
  - "TestNewCommitStatusSeam: do the new cases assert pushes independently of the code's own condition, and do the existing cases' assertions stay as strict as at the review base? Apply the test-economy bullets."
  - "PATTERN.md, pattern/PATTERN-run-fsm.md and pattern/PATTERN-pair-remote.md: does each change state the new rule the code implements, and does the commit message claim anything the diff does not bear out?"
---

Seed focus drawn from the diff `f660ac101..HEAD`, which touches `internal/battencli/commitstatus.go`, its test and three pattern docs.
