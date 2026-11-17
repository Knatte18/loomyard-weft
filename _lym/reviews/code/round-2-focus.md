---
round: 2
focus:
  - "internal/lymtemp/sweep_test.go TestSweep: does a row put a link, made with fslink.CreateDirLink, inside a dead run directory that points at a permkit.Chmod 0o555 directory outside the root, and does it assert after the apply sweep that the outside directory still exists with mode 0o555?"
  - "internal/lymtemp/sweep.go removeTreeMakingWritable and makeTreeOwnerWritable: when the owner-writable pass fails on a chmod or ReadDir, does the sweep still attempt os.RemoveAll and report only its error, and do the doc comments and lymtemp/doc.go say the pass is best-effort, matching the code?"
  - "internal/testkit/enforcement_test.go: does a test pin permkit's stdlib-only import rule the way TestEnforcement_ScankitStdlibOnly pins scankit's, so the rule listed in pattern/PATTERN-test-isolation.md is enforced?"
  - "internal/logger/retention_test.go TestSweep_DeleteFailureTolerance: does it carry a //testtiming:keep line directly above its func line, with a reason naming what it pins?"
---
