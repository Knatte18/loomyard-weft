---
round: 1
focus:
  - "internal/reedengine/remotecontrol.go clientAgentPaneVia: does `list-clients -F` expand pane_id and both pane options against each client's own active window and pane, on every tmux or tmux-compatible binary the repository supports?"
  - "internal/reedengine/remotecontrol.go clientAgentPaneVia: does the `client_name` match hold for the value every caller passes as `--client`, traced from the button binding through `lym orch rc-on`?"
  - "internal/reedengine/remotecontrol.go clientAgentPaneVia: are the row-parsing failure paths correct and truthfully reported, including a listed client whose row has the wrong field count and a field that tmux prints with a tab or newline altered or embedded?"
  - "Does any doc, spec or comment outside the diff still describe this read as `display-message` or state an error text or behaviour the change made false, and do the repository's constraints documents require a doc or test-placement change with it?"
  - "internal/reedengine/remotecontrol_integration_test.go: does the test fail on the old read whichever client attached last, is its setup free of races between attach and read, and does it follow the repository's tier and testing rules?"
  - "_lym/landing/summary.md: does the diff and the current tree bear out its claims, including the old-code failure claim, the follow-up about other `display-message -c` reads and the filed board note, and the `## Merge danger` door and blast radius?"
---

Seed for round 1, drawn from the board entry under `## Goal`, the commit log, the diff and the change description.
The entries are questions to settle, not findings.
