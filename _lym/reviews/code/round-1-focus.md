---
round: 1
focus:
  - "internal/shedarbiter/edits.go and call.go: does readEdits catch every way the arbiter can leave the worktree unsafe (branch switch, rewritten history, uncommitted or untracked changes, detached HEAD, an unreadable log), and does every failure path hand on with cause precondition rather than resume?"
  - "internal/shedarbiter/call.go: is the resume-after-commits rule (a goto is required) enforced on every path that resumes, including a timeout after commits and the agent's own hand-on, and does a hand-on name the commits in both the agent's and the call's own question?"
  - "internal/loomcli/arbiter.go and internal/loomengine/arbiter.go: is the Worktree seam wired through fabric's read-only reads only, and left unset for Batten, so no lym code runs a mutating git command?"
  - "internal/sheddriver/drive.go, drivehandback.go and internal/shedrun/settle.go: do the commits reach the re-step settle record and the stop report's attempt line, and does the new commits key keep existing records and parsers compatible?"
  - "contracts/stencils/loom/loom-template-arbiter.md and the two arbiter fragments: against the Goal's three questions (what it may edit, how an edit is committed and checked, how edits show), does the stencil wording keep the forbidden actions (push, amend, rebase, reset, branch switch, edits under _lym/ or .lym/) and defer the plan, record and PATTERN changes to the repair verbs?"
  - "Test economy: TestPrompt is reported as contained, covered by TestCall_Edits. Compare their assertions before filing, and check that every behavior the diff adds has coverage."
  - "Summary claims in _lym/landing/summary.md against the diff, notably the claimed trailer, the claimed 'Go enforces' checks and the PATTERN no-change claim; also whether docs/overview.md and the doc.go files carry the change."
---

The decision record holds only the board entry under `## Goal`, so the entry's three questions are the measuring stick, and the record's other sections are empty.
