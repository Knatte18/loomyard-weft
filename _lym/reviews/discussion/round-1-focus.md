---
round: 1
focus:
  - "Goal and Scope against the board entry `keep-landed-records`, its `Decision for` notes and related entries: does every record the entry asks to keep have a Decision that keeps it for every way it is produced? Check parent answers and review-round fixer reports in particular."
  - "D1: trace every consumer of `statusCommitPathspec` and `loomengine.PathsOutsideRunRecords`, and every other committer, cleaner or seeder of a run's `_lym/`. Does the new directory behave as stated when it is absent, empty, ignored, holds temp or lock files of an atomic write, or is written between the last status commit and teardown's tag?"
  - "D2: does the premise hold that every PR-Review findings file passes through `lym loom pr-findings` and that the verb already commits on both the send-back and the fix-here path? What does a refused, failed or retried call leave in the shared sequence?"
  - "D3: does the live parent-review ask record have one locked update path that every write goes through, from processes that can resolve the worktree's tracked directory? Does a later round, a reset or a new run drop entries or reuse ask numbers so that a whole-file or per-number copy overwrites history?"
  - "D4: can the single scratch report path hold an earlier attempt's report when a session ends without writing one, and can the attempt record's numbering restart (re-step, resume of an interrupted session, reset, a second row) so that one copy overwrites another?"
  - "D5: for each landing mode and each way Finalize can resume, are the landing commit, its message, committer time, parent branch, PR URL and drift verdict durably available and stable? Does the commit that carries the row happen before the tag, and is the commit message still the Describe text as merged?"
  - "D5 and D6: which archived tags are not landed runs (cancelled, abandoned, diverged teardown), and what does `lym report landed` list, order, derive and extract for them? Check the premise about what `reportengine.Open` enumerates."
  - "D6 attack surface: what can extraction from a tag tree write or follow (symlinks, paths leaving `<dir>`, an existing file or link as `<dir>`, a partial write after a failure, a `--tag` naming another slug or a non-archive ref), and is each bounded?"
  - "D7 and PATTERN changes: verify that no other code path deletes or overwrites an `archive/` tag locally or on origin. Does the new `PATTERN-run-records` text keep what the current entry requires, and does the retention rule sit in an entry whose subject covers it?"
  - "Facts the design rests on about git and existing code (teardown commits uncommitted records before the tag on every path, atomic writes versus a concurrent `git add`): is each stated with its observed result and where it is pinned?"
  - "Open risks: check `pr-modes-no-agent` and `describe-review-and-sync` and the runs now in flight. Does either overlap leave a question only the parent can settle, or can the record settle it itself?"
---

Seed for round 1.
The record rests on existing write, commit and teardown paths, so the premises about those paths come first.
