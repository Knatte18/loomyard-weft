---
round: 1
focus:
  - "internal/beamcli/hubstop.go: does the sequence follow D9's order (pause and confirm, pairs through reed down, BeamDown for prime and beams, beam reed sessions, Reap, own Lift, prime reed session last), and does a failure or unconfirmed idle at one step still let later steps run?"
  - "internal/beamcli/hubstop.go: is the detached re-start spawned through selfexec with a logged spawn as fabricengine does, and does the re-started process avoid re-spawning itself?"
  - "internal/hibernateengine/watch.go: does the exported pause-and-confirm own only an entry it added and leave a standing hub entry (operator's or hibernate's, or a Bolt hold) untouched, and does the carrier behave as before?"
  - "internal/orchcli/stop.go: does BeamDown end the orch's registered Claude process through runproc.EndSubject as its last step for every caller (beam stop, cancel, pawl row), including the caller inside that orch's own session?"
  - "internal/reedcli/up.go: does the pair end accept a pair a hub pause covers, and do other callers of reed down keep their prior refusals?"
  - "reed templates (posix and windows) and config_test.go: is the prime-group stop entry identical in both with the confirm text D9 names?"
  - "Scope: does the diff touch anything outside the five packages D9 names, or leave out a D9 delivery (no tracked record, no per-run intent, no dobbyengine, machinecli or termwindow code)?"
  - "Repository rules: PATTERN-child-processes, PATTERN-cli-cobra, PATTERN-refusal-way-forward and PATTERN-test-only-code against the new code, and whether doc.go and docs/overview.md updates match the change."
---

Seed focus from the decision record's D9 and the diff stat; the redundancy report lists no contained test.
