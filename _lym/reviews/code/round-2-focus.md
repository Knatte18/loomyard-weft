---
round: 2
focus:
  - "In the 'Making a beam' section of contracts/orchrole/orch-role.md, settle whether the way the section tells a task run of a project from a beam is sound: batten's own rows (Worktree-Create through Run-Shed) are the same for both, so the test must rest on the entry's kind and on something the inner run actually reports, and must be checked against what `lym loom status` reports in a beam worktree."
  - "In the same section, settle whether step 2's description of `--anchor` matches `lym beam create --help` (internal/beamcli/cli.go), including that the path is relative to the code repo root and tracked in the commit the beam forks from."
  - "In the same section, settle whether any instruction to read `lym batten status` is tied to a trigger, given that item 4 of The run loop in the same file says never to read it on one's own to check on runs."
  - "In the same section, settle whether the claim about what starts a beam's run is no stronger than internal/beamcli/cli.go and internal/beamcli/doc.go support."
---

The focus lists the places the round 1 fixes touch; it does not cap what the reviewer may find elsewhere in the diff.
