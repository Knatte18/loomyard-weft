---
round: 2
focus:
  - "In `_lyx/discussion/decision-record.md`, the Decision that introduces the landing config key `publish_verify` (D4): does it state what the key can run, who can change it, and what bounds a task-edited value? `landing.yaml` is the task worktree's own tracked `_lyx/config/landing.yaml`, so a task's diff can empty it or replace it with any shell command, and `verifyrun` runs the value through `sh -c`. Settle whether the Decision names that bound, or accepts the gap explicitly."
  - "In `_lyx/discussion/decision-record.md`, the Decision that introduces the `//lyx:guard` marker (D5): the round gate runs a repo-scanning test only when it is marked. Does the Decision state what an unmarked scanning guard lets through and what bounds it, which build tag a marked test runs under, and does an acceptance criterion require the existing repo-scanning guards to carry the marker, with a stated way for the plan writer to find them?"
  - "In `_lyx/discussion/decision-record.md`, D2 (round-gate command order and impacted set) against D1's gate table: does every row and every bound agree on which build tags (`integration`, `tmux`, `llm`) each gate compiles, and does D2's closing bound name what a tagged-file change lets through the round gate?"
  - "In `_lyx/discussion/decision-record.md`, D6 (comment line-break rule) against D7 (the lint) and D11 (the kept PATTERN line): do they agree on whether untouched lines of a touched comment block are flagged or rewrapped, and on which line ends count as sentence ends, including a colon and a terminator followed by closing punctuation?"
  - "In `_lyx/discussion/decision-record.md`, any edit made to the Decisions this round: does each new or changed guard, escape hatch or fallback state what it can skip or let through and what bounds it, and does any content moved out to `support-log.md` leave its requirement or constraint in `decision-record.md`'s Decisions or Constraints, since Plan-Write never reads `support-log.md`?"
---

Round 1 was not converged, so round 2 re-checks the sites above from the current text.
