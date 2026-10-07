# Round 2 fixer report

Verdict was APPROVED with five findings; all five are fixed in `decision-record.md` and `support-log.md`.
The section structure of both files is unchanged.

## Changed

- **F1** (MEDIUM, design): Decision 3's bound now says it covers a waiter only, that a holder's length is unbounded because `gitexec` sets no command timeout, and that a hung status push now also blocks detached children behind it, accepted.
  Open risks replaces "until git gives up" with the same statement and adds the entry for a status push that stalls in git while holding the lock.
  The support log's rejection of a no-timeout wait and Q7 are narrowed to what the bound buys.
- **F2** (MEDIUM, design): Decision 1 now makes every error of the new entry name the failed side and its worktree path and wrap the side's error with `%w`, so `errors.Is` holds for one side and for the join; `PushAnchored` keeps its unwrapped sentinel.
  Decision 2 states the envelope and warning carry the side-named text and that the envelope does not say the records were unpushed unless the records side failed.
  Acceptance criteria gain the code-side-only case.
  Q8 is updated.
- **F3** (MEDIUM, decision): Decision 1 gives the entry the lock wait as an input; Decision 3 scopes the bounded wait to the seam and `commit-records`; Decision 5 decides `lyx fabric push` normal mode waits without limit, like the repo-root lock it replaces, and that the detached child's `push` bypass keeps `CoalescePushBothAt`.
  Acceptance criteria gain `lyx fabric push` against a lock held by a detached child.
  Open risks notes detached children and `push` wait behind a hung holder.
  New ledger entry Q11.
- **F4** (MEDIUM, decision): Decision 2 now names the `commit-records` way forward: a rejection that stays names merging the remote branch in the worktree the error names (or clearing a remote rule's objection), then re-running; any other failure stays transient; the spec row splits in two as the Publish rows do, with no denied command.
  The `PATTERN-refusal-way-forward` constraint now points at those two rows and a test reaching each, and Acceptance criteria add that test.
  New ledger entry Q12.
- **F5** (NIT, consistency): Scope Out and Decision 4 now name `PushAnchored` and the entry instead of a shared primitive.

## Deferred

None.
