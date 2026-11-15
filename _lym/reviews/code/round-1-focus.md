---
round: 1
focus:
  - "internal/orchcli/start.go launchOrch: does an orch start whose resume is refused for a live holder end the holder only when runproc registers it as this worktree's orch, then resume the same session id, and never fall back to a fresh launch over a live holder on any path?"
  - "internal/orchcli/start.go: does a holder lym has not registered, including a pre-upgrade orch process, get refused with its pid and a way forward (exit that session, then lym orch start), and does a test reach that refusal?"
  - "internal/orchcli/start.go and internal/shuttleengine/claudeengine/resume.go: is the fresh launch with the carry-over reached only when no local transcript exists, does the start say so, and is no tracked record of the orch session id written (D10)?"
  - "internal/runproc/record.go and end.go: is the orch role keyed by the worktree, does its registration outlive the strand's removal and get pruned when the process dies, and does the new end-everything-for-a-subject function end run strands, step loops, the orch strand and its Claude process, with the dobby last, with failure paths handled?"
  - "internal/orchengine/owner.go and internal/orchcli/watch.go: does the orch watcher call the new subject-ending function for a subject owned elsewhere, and does every new function have a production caller (PATTERN-test-only-code)?"
  - "internal/shuttleengine engine.go and run.go: does the carry-over change match D9 and D10, and is anything in internal/fabriccli/anchor.go changed beyond what the record licenses (scope)?"
  - "pattern/PATTERN-child-processes.md and the doc.go files: does the pattern change match the constraint that the orch's Claude process registers in runproc and D9's ends go through it, and do the docs of changed packages match the code?"
  - "Test economy: check TestStart_StarterErrorFailsAndSavesNoState, listed as contained, against the assertions of TestStart_ResumesTheRecordedSession and TestStart_ResumeOverALiveHolder before filing anything."
  - "internal/fabriccli/anchor_test.go and the commit messages: does the diff to anchor.go and its test have a basis in the record, and does any Pattern-Departure trailer name a real entry that forbids the answer key's way?"
---

The record has no plan, so the amended board entry under `## Goal` and D9 and D10 are the measuring stick.
The focus lists where the diff makes D9's behaviours and where it may reach beyond them.
