---
round: 2
focus:
  - "In `gateProcessLive` in internal/battencli/wire.go, settle whether a live fixer-role pane-process record under the pair's slug can keep counting as a live wait after the fixer's run has ended without a done finish, and whether the bound the code applies to such a record comes from the fixer's own timeout configuration rather than a literal."
  - "In `TestReadAgentActivity_ReadsLiveRunsAndTheWaitMarker` in internal/battencli/wire_seams_test.go, settle whether a gate or fixer record registered under a different owner name than the run's worktree name is shown to leave `readAgentActivity`'s waitLive false."
  - "In `TestRun_Wait_Classification` in internal/shuttleengine/wait_test.go, settle whether the run-end stamp is asserted for every finalized outcome (done, died, timeout, api-error), not only the cleaned-up done row, against the contract that every finalize stamps."
---

Each entry names a site and the question to settle; none limits severity.
