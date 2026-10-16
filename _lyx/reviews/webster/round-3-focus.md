---
round: 3
focus:
  - "Tmux integration tests in internal/reedcli and internal/reedengine that create strands: with strands placed in own windows, does any remaining test assume a single shared window, a hardcoded window index, or a pane_top ordering across windows?"
  - "internal/reedengine/ownwindow_integration_test.go, the DeadPaneLeftInTheShellWindow subtest: with the leftover strand given a relaunch command that stays up, does Resume of a moved dead own-window pane yield a live strand in its own window with a fresh pane id, or does lifecycle.go or spawn.go have a defect?"
  - "Tests that type into or paste to a freshly created own-window strand pane: do they wait for a readiness signal from the pane's shell before sending input, given that own-window creation returns before the pane's command has run?"
---

Targets the failure classes behind the Publish verify failures; fixes are expected to be test-side unless the Resume check shows otherwise.
