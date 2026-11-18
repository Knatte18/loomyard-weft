---
round: 1
focus:
  - "Attack-surface, D8: the `lym-view/<branch>` force-push is bounded to `lym-view/` refs. Settle whether the Decision states what a stale or concurrent view, or a view of a branch whose `parent_branch` is wrong, lets a pull request merge."
  - "Attack-surface, D9: records paths are stripped from tree identity, dirtiness and held work. Settle whether a change under a path named `_lym` or `.lym-anchor` that is not a records file can pass gates, verify or the landed check unseen, and what bounds it."
  - "Attack-surface, D5 and D7: step 3 deletes and restores paths by each side's anchor marker, and the prime push guard admits commits carrying the merge trailer. Settle whether a forged trailer or a marker edited on the source side can skip the strip or the guard, and whether the Decisions bound that."
  - "Attack-surface, D2 and D6: `--single-repo` and the tolerated missing view ref on teardown. Settle whether the stated bounds cover a wrong-repo invocation and a view ref deleted by someone else."
  - "Unpinned tool fact, D4, D5, D8: check each claim about git behavior (`git commit -- <paths>` leaving other staged code, merge without commit then path restore in index and worktree, merge-base as the view parent, GitHub deleting a merged head, fast-forward carrying records). Settle whether each gives an observed result and a pin: a test, a recorded command and output, or documentation."
  - "Deferred research, D2, D7 and Open risks: check that no Decision, Constraint or risk leaves a fact the design rests on for Plan or an implementer to find, such as how a replay recognises a records-only commit, or what makes a commit 'carry fabric's merge trailer' when it is read back. Do not count D9's 'Plan enumerates the readers' as one: call-site enumeration is Plan's."
  - "Consistency across D1, D3, D4 and the Constraints: the marker is tracked and read by `lymcwd`, written only by fabric, rewritten in an anchor change, and never excluded. Settle whether the Goal, Scope, Decisions, Constraints and Acceptance criteria say one thing about who reads, who writes and what is excluded."
  - "Consistency between the record and the support log: Q6 is open in the question ledger and is in Open risks. Settle whether that is a `decision` finding for the parent or already settled, and whether the Auto-mode assumptions list matches the auto-picks the log records."
  - "Relocation, before any gap-filling: apply the completeness-before-leanness test to D2's docs sentence, D6's mechanics narrative, D1's Rationale and the Constraints' PATTERN-background paragraph. Settle whether each carries a requirement the plan writer needs or is deliberation that belongs in `support-log.md`."
  - "Acceptance coverage: each criterion is behavior a test can check, or opens with `Review:`. Settle whether D4's lock sharing, D6's owed-push hold and D7's prime wait have a checkable criterion."
---

Findings on the board-entry edit and task-filing bullets: no Decision edits the entry's body or files a task, so no focus entry is spent there.
The rubric's `Do not flag` list is respected: no entry asks for a notes subsection, rejected alternatives in the record,
or a full call-site enumeration.
