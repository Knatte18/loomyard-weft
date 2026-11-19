---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
---

## Changes

- F1, commit `5d36f7e7`: `internal/loomcli/prfindings_integration_test.go` now runs the verb through `clihelp.ExecuteIn(Command(), <pair worktree>, ...)`.
  It writes a pull-request record and a BLOCKING findings file under the worktree's `.scratch/`, and asserts the ok envelope with `send_back: true`.
  It keeps the tier-1 gate, the stencil seeding and the tapestry recipe.
  The helper-level test is deleted and the test is renamed `TestPRFindingsCLI_SendsBackOnATapestryRun`.
- F3, same commit: the test copies `hubforge.Shape{Anchor: ".", Pairs: []string{hubforge.TemplatePairSlug}}`, with no slug constant and no `AddPair`.
- F2, commit `74fa4dfb`: the two comment lines in `internal/loomcli/sharedbootstrap.go` now say the building paths and pr-findings' send-back edge pass through,
  that status and pause never do, and that the edge's process exits right after it reads the shed, so its watch is never waited on.

`lym gate test --tier 1 ./internal/loomcli` passed, and the new test ran.

A later gate found the test file's second header line over the line-length limit.
It is now split at its colon, in a further commit, and the gate reruns on that.

A later gate reported that no mapped test reaches `loomCLI.prReviewSendBackEdge` or `loomCLI.wireLanding`.
`TestPRFindingsCLI_SendsBackOnATapestryRun` already reaches both through the verb, and the tier-1 gate for `internal/loomcli` passes with it running, twice.
No test was added: the reach map holds the package's tests as recorded before this round's files changed, so a second test would repeat the same behaviour.
No code changed in this repair.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None.
