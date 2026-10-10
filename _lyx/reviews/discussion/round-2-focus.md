---
round: 2
focus:
  - "In decision-record.md D1 and its Open risks and acceptance criteria, settle what outcome a gated run ends with when its output files exist and a held shell never reports completion by the run deadline; check it against classifyDeadlineExpiry in internal/shuttleengine/wait.go and PATTERN-completion-signal (a negative answer is finalized only after allOutputFilesExist is consulted first)."
  - "In decision-record.md D6, settle whether the build-tag environment set the classifier evaluates is stated so that unix and the GOOS-implied tags are derived from GOOS and not assigned freely; check that //go:build !windows beside //go:build unix and //go:build unix beside a _windows.go file each get the verdict D6's bound ('only an answer whose candidates provably never coexist passes') requires."
  - "In decision-record.md D3, settle which slices contractFileStatus in internal/websterengine/contractevidence.go reads (Master writes, fork writes, recovery writes) and whether a recovery write to outcome.yaml later than Master's last write keeps the finding standing; check the Bound sentence and the acceptance criteria against that code."
  - "In decision-record.md D3, settle what widening the reset's own-path set (ownTrackedPaths, ResetDeps.OwnPaths) lets a reset discard without the dirt refusal, and whether the Decision states that bound and an acceptance criterion covers the reset side."
---

Round 1's gating findings were all in D1, D3 and D6; the entries above keep the next reviewer on those sites without capping severity.
