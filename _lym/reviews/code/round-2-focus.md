---
round: 2
focus:
  - "internal/landingshed/finalize.go, gateUnlocked and landUnderLock: before any merge, conflict session, gate, fixer, fix review, commit or push in the lock-free pass, is the run's owner and origin state judged (PATTERN-owner-machine), and are the checks under the lock still run? Check the skipsCheck doc in internal/loomcli/prepush.go against the answer, and whether a test reaches each halt."
  - "internal/landingshed/finalize.go Call and internal/landingshed/parentalign.go parentMergeTip: does every parent-lock acquire on a pass after a lost race or a fixer queue at the front, including the tip read taken when the push is skipped or there is no upstream, and what does ParentAlign's own call pass?"
  - "internal/landingshed/finalize.go, the held read and the gatedTip compare under the lock: can a change in whether the parent already holds the task's work, or a task-branch fast-forward during the owner checks, count a lost race that was none, or skip a compare that was needed?"
  - "internal/landingshed/finalize.go and deps.go, the config-change notice: which merge-base is it read against once the lock-free pass has merged the parent, and do the comments and the ConfigChanges doc state that?"
  - "internal/landingshed tests for the lock-free pass: are a parent behind its upstream, a failing gate with fixer and fix review, the new error and halt paths, front-of-queue re-acquire and the skipped verify under the lock each pinned by a test that would fail on a regression?"
  - "contracts/recipes/*.yaml comments and the type and freeLock docs in internal/landingshed/finalize.go: does each describe the order of merge, gate, fixer and parent lock that the code has?"
---
