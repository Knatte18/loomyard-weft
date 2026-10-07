---
round: 2
focus:
  - "Check the Windows behaviour of the verify-shell kill in `decision-record.md` (the Decision on `verifyrun`, the Constraints bullet on Windows, and the Acceptance criterion about surviving processes): does every one of them promise only what a new process group plus `os.Process.Kill` delivers on Windows, given `verifyrun` may import only the standard library and `internal/logger`?"
  - "Check the Decision that sets a done-when list on a shuttle strand so reed's resume drops finished ones: for a gated run (`handleGatedBoundary` in `internal/shuttleengine/wait.go`) whose outputs exist but whose gate is unpassed, does the Decision state what is dropped, what still re-prompts the agent, and what bounds the change?"
  - "Check the batten verb Decision that resumes a halted run: does it state, for every run state (done, awaiting, running with a live driver strand, running under a Go driver holding the run lock, halted, missing status file), exactly one outcome, and does it say what happens with an unfinished merge and the bootstrap lock that `lyx loom start` already honours?"
  - "Check the wait-mark Decisions: for a mark left on a surviving pane after a crash, and for a landing callback whose driver strand is gone or replaced, does the record state what the mark shows, what clears it, and that a failure to set it never fails the verify?"
---

Each entry names a site and a question; none caps severity or pre-states a verdict.
