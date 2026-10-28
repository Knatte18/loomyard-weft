# Support log — No test is skipped by design

## Interview

A darn run holds no interview; the decisions are the writer's own, read from the board entry and the code.

## Rejected alternatives

- Building a real fabric for `TestBuildLoomShed_OutputShape` (D3): it moves a pass-through check to `tier1` for behavior the integration tiers already reach.
- Splitting every OS-flagged table into per-OS files (D2): it duplicates a table for one or two rows.
- Keeping the matrix omissions as skipped subtests with a log line (D4): the board entry rules out generating them.
- Leaving the always-skipped verify-gate subtest in place until the key carries `--base` (D6): it stays a skip by design until then.

## Review rounds

_No rounds yet._

## Question ledger

- Is a root-user skip a missing prerequisite? Self-pick: yes, it stays.
- Delete or fix `TestBuildLoomShed_OutputShape`? Self-pick: delete.
- How do OS-bound table rows avoid a skip? Self-pick: filter them before `t.Run`.
- Is the unconditional verify-gate skip in scope? Self-pick: yes, delete the subtest and file a board note.
