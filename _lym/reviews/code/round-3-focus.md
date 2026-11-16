---
round: 3
focus:
  - "Verb names in internal/reedengine/panebin.go (composeRemoteControlClickCommand, composeRemoteControlStateCommand) and internal/orchcli/remotecontrol.go: is each of the rc-on and rc-bar verb names declared exactly once, with no repeated string literal left in either package?"
  - "_lym/landing/summary.md against internal/orchcli/remotecontrol.go (turnOnRemoteControl, recordRemoteControl, remoteControlBarWordAt) and internal/orchcli/start.go (startOrch): does every claim about the click on a lit record, a Codex orch's button and the button's visibility match what the code does?"
  - "internal/orchcli/start_test.go TestStart_LiveStrand: does every row reaching the success path assert the RC word, including the rows that adopt an already-running strand with a live watcher, with one row keeping an already-lit record for the same strand?"
---

Round 3 re-checks the three open findings' sites after their fixes.
