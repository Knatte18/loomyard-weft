---
round: 1
focus:
  - "Attack-surface, D3: the per-copy fingerprint walk is removed from `CopyHub`. Settle whether the Bound (stray `SharedHub` write reaches earlier copies) is the whole of what the walk no longer catches, including a registered template's build function or a test writing to a copy's shared pack files."
  - "Attack-surface, D2: a new registration verb with a `testing.TB` that refuses `Cleanup`, `Setenv` and `TempDir`. Settle whether the Bound covers what a build can still do that escapes the template root (absolute paths, env, global git config) and what a duplicate-key or failed-build lookup lets through."
  - "Attack-surface, D1 and D5: the porcelain fallback and the old hook that stays in installed hubs. Settle whether each states what it lets through and what bounds it."
  - "Deferred research, Open risks: the line on git 2.43.0 at the floor leaves a fact D1's parity rests on (ordering and prunable marking) to a later parity test. Settle whether the design should instead record the answer with evidence from the floor version's manual or release notes."
  - "Deferred research, D2 and Open risks: the claim that concurrent builds no longer queue, and the 'pays from its third use' figure. Settle whether each is a recorded measured fact or a fact left for an implementer to find out."
  - "Board-entry and task filing: check each Decision, D9 and D10 for any change to this task's own board entry body or scope, and for any follow-up filed or promoted as a task rather than a note. The D10 re-measure note and the split's darn tasks are the sites to read."
  - "Parent-decision class: D3 departs from the entry's 'no amend and push --force per copy', and D5 adds `internal/adoptcli` sites. Settle whether the entry's text contradicts either so only the parent can settle it, or whether the writer's recorded pick stands."
  - "Relocation: read Decisions, Rationale lines and Open risks for deliberation narrative or measured figures that belong in `support-log.md`. Apply the completeness-before-leanness test first: a requirement or constraint the plan writer needs is extracted into Decisions or Constraints before anything moves."
  - "Consistency: D9's task lines against D5, D6, D7 and D8 (which task owns which file and test), the Scope list against the Decisions, and each Auto-mode assumption against its Decision."
---

The record is long and carries one lens review already logged in the support log's Interview.
Round 1 should weigh the attack-surface and deferred-fact bullets above first, since D2 and D3 weaken or add guards and D1 leans on a git version.
