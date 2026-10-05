---
round: 4
focus:
  - "Every refusal the diff adds or changes in webster, shed or loom, including those reached only through a generic wrapper such as `refuse(\"%s\", err)` or a Bouncer Stuck Reason: does its message carry a `way forward:` clause, does `contracts/specs/refusal-spec.md` have a row for it, and does a test assert the clause?"
  - "`internal/loomcli/decision.go` decision add: the commit-failure refusal and the missing `## Decisions` heading refusal; confirm each has its own message clause, spec row and asserting test."
  - "`internal/shedadapters/bouncer.go` circling decision handling: the unreadable-decision and already-settled-accept Stuck Reasons; confirm each names a way forward that actually re-enters the run, has a shed spec row and an asserting test."
---

The previous fix commits touched exactly these sites; a fresh reviewer should also sweep for sibling sites that reach a refusal through a shared helper.
