---
round: 2
exclude_lenses: []
focus:
  - "Resume paths in `_lym/discussion/decision-record.md` D1, D2 and D10: for `lym s` and `lym adopt --here` run after an adopt worker died or stopped after the `owner` step but before `deploy`, `beam` or `check`, does the record decide that the unfinished record is read, the adopt resumes at its first step not done, and no `beam start` runs on the old lym until the adopt ends? Settle it from the decisions and their acceptance criteria."
  - "D4's covered worktrees: during `lym adopt root`, does the decision state what the orch watcher and the dobby of a beam with no owner of its own (which reads its parent's owner through `hubgeom.ParentOf`) do, so neither stops, exits, archives nor starts a pass before the subject's owner write stands?"
  - "Race and staleness bounds in D1 and D2: `lym adopt --discard` against a concurrently starting worker and `adopt.lock`; a resume whose first step not done is `owner` or earlier, against a third machine that adopted the subject since; a direct call of the hidden worker verb skipping caller validation; a caller whose worker lost the lock reading an older done record. Does each Decision state what it can now skip or let through and what bounds it, and does the bound hold against the code?"
  - "D10 and D11 against existing code and `PATTERN-owner-machine`: what `beam start` does when origin is unreachable (`orchCLI.prepare` warns only); whether the archive-tag readers in `internal/reportengine` (`selectRuns`) and `internal/fabricengine` read D11's tags of live runs and of `root` and beam slugs as archived runs; and whether the rule text of `PATTERN-owner-machine` bars the old owner's archive-tag push, with the precedent the decision cites."
  - "D12's `--archives` listing: which tags does it list, given that teardown and `--drop-local` write the same tag shape, and does it name the repository (code or weft) whose origin it reads and the branch it compares each tag with?"
---

Reviewers run all four lenses again, because every finding of round 1 touched a decision the fixer will rewrite.
No lens is excluded: the interactions lens raised the BLOCKING F1 with the walkthrough lens, so its area is not settled.
