```yaml
lens: fix-proposal
outcome: reported
```

Findings: one LOW. F3 and F4 hold up.

- **LOW** — `contracts/stencils/darn/darn-fragment-make-change.md:17` ("When `refuse_raw` lists it, report the skipped tests to your parent."): a darn writer may have no parent.
  - `darn-template-write.md:50` and `:54` say "With a parent …" and "With no parent …", and `parent_directive` may be empty (`darn-template-write.md:7`).
  - The new instruction gives a parentless writer nothing to do.
  - A stopgap fits the existing pattern: "report the skipped tests to your parent, or, with no parent, name them in the change description".
  - `webster-body-implementer.md:117-119` is unaffected, since Webster always has a parent.

- **F1** (hand run only while `refuse_raw` does not list `dotnet test`, otherwise report to the parent): I built no breaking case beyond the LOW above.
  - The new `noTestNotice` text, the `TestRunTierNoTestNotice` expected line, the stencils and the doc.go sentence all agree.
  - The "never a raw test command" rule still holds, with the hand run as its named exception.
  - A reader cannot see `refuse_raw` from the agent session. The guard's refusal notice still tells them, so I don't count this as a defect.

- **F3** (rows for a failing invocation and for no targets): both rows work and I found no breakage.
  - The `case "{targets}"` quoting leaves the existing rows' matching unchanged.
  - The `*bad*` arm precedes the default arm, so the failing row reaches `exit 3` and `runInvocations` returns before `noTestNotice`.
  - The "{dir}" placeholder is replaced on both the expected stderr and the log check.
  - The gate run passed.

- **F4** (a test of `CodeLocation` with the code directory absent): the fixer placed it in a new `fabricengine` unit test, one of the two placements the review allowed.
  - The `NoCodeDirectory` row fails if the stat guard is removed, because `NamedWorktree` would then run on a hub with no such worktree and return an error or a different location.
  - The pointer-identity check proves "unchanged".
  - `NotAWeftSibling` is a second branch, not a duplicate.

Calls made:
1. `grep -rn "parent" contracts/stencils/darn/*.md`: to check whether a darn writer always has a parent. My context didn't show the darn template.
