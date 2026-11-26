---
round: 2
focus:
  - "internal/loomcli/sharedbootstrap.go ensureStatusStrand, its callers in internal/loomcli/start.go and internal/loomcli/wiring.go, and restoreStatusPaneBeforeWake in internal/loomcli/resume.go: when adding or replacing the status strand fails, does resume's envelope name it in status_pane with how to restore the pane, and does start treat each failure as the answer key asks?"
  - "internal/loomcli/smoke_drivingsurface_test.go TestSmokeStatusStrandAcrossDriverSeeds and internal/loomcli/resume_test.go TestResumeVerb: does a test fail when the real restore closure in internal/loomcli/wiring.go is dropped, and does it check that a stale status strand ends up replaced by one live strand?"
  - "The file header, doc comments and warning texts of internal/loomcli/sharedbootstrap.go, the Short and Long of resumeCmd in internal/loomcli/resume.go, and internal/loomcli/doc.go: does each match which verbs call the status-strand helpers and what resume does?"
  - "_lym/landing/summary.md, its claims about resume and a viewer's tmux window and about a failed restore: does the diff bear each out, including the case where the replaced status strand's window is the one the viewer is on?"
---
