---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
---

# Round 1 fixer report

## Changes

Commit `a1e591ec3` (F1, F2, F3 share `remotecontrol_test.go`):

- F1: `clientAgentPaneVia` in `internal/reedengine/remotecontrol.go` skips only other clients' rows, and returns `reed: client %q has an unreadable list-clients row` for the clicking client's row with a field count other than 4. The `TooFewFields` row expects that text.
- F2: `OtherClientsRowsAreSkipped` lists `/dev/pts/30` before `/dev/pts/3`.
- F3: `AgentPaneWithoutTheMark` ends its row in `\r\n`.

Commit `8bd656c3a` (from the self-check):

- `clientAgentPaneVia` skips empty lines, so an empty client name is reported as not attached and not as an unreadable row. No test added: it is not a coverage gap of a review finding.

`lym gate test ./internal/reedengine` (tier0) exited 0 with no failures after each commit. The tier-2 integration test was not run: the changes touch only row parsing and its tier-0 test.

## Self-check

- fix-proposal, LOW, empty `client` matches the trailing empty line and reads as unreadable: fixed with an own change (the lens proposed none).
- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-affected-set: no findings.

## Disputed

## Deferred

## Rulings
