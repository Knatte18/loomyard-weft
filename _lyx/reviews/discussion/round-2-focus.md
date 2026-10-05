---
round: 2
focus:
  - "decision-record.md D5: for every time seam the Decision allows, settle whether a test can shorten it per instance and still call t.Parallel under PATTERN-test-parallel and PATTERN-test-time-seam, or whether it writes process-global state."
  - "decision-record.md D6: settle whether the Decision states what the retag from integration and smoke to tmux lets through (cardGateCommand's -tags integration run, plan verifies running go vet -tags smoke or -tags integration), and what bounds it, so no test ends up under a tag no gate runs."
  - "decision-record.md D2: settle whether the Decision says how a test whose run spawns the built lyx binary or another subprocess is classified when per-test coverage cannot see the child's blocks, and whether that classification cannot send a prune task toward deleting the only test of a path."
---

Entries restate the sites they depend on and do not cap severity.
