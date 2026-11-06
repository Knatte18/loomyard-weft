---
round: 1
focus:
  - "Rules dropped silently: compare `git show b30cccce:contracts/orchrole/orch-role.md` against the new `orch-role.md` and the four `role-*.md` files, passage by passage. Does every binding rule survive in force and meaning, or is any weakened, merged away or gone, including the rule that the orch uses only the commands open to it?"
  - "Where the old role's passages disagreed, does the new text follow what the code does? Check each such passage against the code it describes."
  - "Moved passages: does the role carry one line saying when to read each on-demand file (beams, landing halts, intake, hibernate, Slack)? Is each file rendered beside the role and copied under `.lym/orch/` by the code in `contracts/orchrole/orchrole.go` and `internal/orchengine/prompt.go`, with no other session's stencil changed?"
  - "Removal of the rendered command index (`internal/orchcli/index.go`, the marker, `watch.go`, `watcher.go`): is anything still reading or rendering the removed marker or index, and do the tests that remain still cover the start, resume and watch behavior? Does lym's behavior change beyond what the goal licenses?"
  - "Change description `_lym/landing/summary.md`: does it list each dropped and each moved passage with its destination, and state start cost in tokens before and after? Check each claim against the diff."
  - "`pattern/PATTERN-stencil-ownership.md` and the stencil registry and marker tests: are the edits consistent with the new files and with each other, and does any new test assert static role prose?"
---

Round 1 seed. The decision record carries only the board entry under `## Goal`, so the amended entry is the measuring stick.
