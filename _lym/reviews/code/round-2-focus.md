---
round: 2
focus:
  - "internal/battencli/arm.go fresh-start seeding (`seedFreshStatus`/`seedAbsentStatus`) and internal/battencli/commitstatus.go `newCommitStatusSeam`: settle whether a fresh batten run sends Worktree-Create through the commit seam so the push happens once there and Seed-Child through Run-Shed take the pushless start core, and whether a test drives that production path rather than a synthetic call."
  - "internal/battencli/commitstatus.go start branch: settle whether a start-row push that failed with a non-owed error, or was cut off by a crash between commit and push, is retried when the run resumes, or whether the rest of the start runs pushless until Land."
  - "internal/battencli/commitstatus_test.go `TestNewCommitStatusSeam`: settle whether removing any of these makes a row fail: the marker write in the start core, the Worktree-Create exclusion in `isStartRow`, the pre-Run-Shed producer conjunct, the start core's omitted gate admission."
  - "PATTERN.md and pattern/PATTERN-pair-remote.md, PATTERN-run-fsm entry, and internal/statuscommit/statuscommit.go `Deps.Push` doc: settle whether the start-row exception reads the same across every entry and doc that states when a transition pushes."
---
