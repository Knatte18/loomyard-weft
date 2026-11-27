---
round: 2
focus:
  - "`runRule`, `outcomeClassOf`, `fieldReadError.Error`, `fieldReadError.Unwrap`, `snapshotReader.read`, `readLocalRefs` and `Refusal.Error` in the fabric packages, with `testOnlyCodeAllowed` in cmd/lym/testonlycode_linux_test.go and pattern/PATTERN-test-only-code.md: how do the code and the PATTERN text stand against each other at HEAD, given the decision record's added entry of 2026-11-27 on chain parts leaving production code for a later part to call?"
  - "internal/fabricengine/executor_test.go `TestRunRule` against the loop in internal/fabricengine/executor.go: do the rows pin how a transport failure on a re-read, a joined error mixing transport and other failures, a phase requested twice, and a nil lock are handled?"
  - "internal/fabricengine/snapshotread_integration_test.go against internal/fabricengine/snapshotread.go: do the tests pin that one failing store stops no other, what the records-store field returns for a readable and an absent store, and how a tag naming no commit and a prunable records worktree are read?"
  - "Deleted and trimmed tests across the range, the list in the body of commit f4fd75355 included: does each removed assertion have either a holder in another test file, on the same behaviour and no looser, or a `Loosened-Check:` trailer naming a measured cause?"
  - "Any test the commits of the range add: does it duplicate coverage an existing test holds, or compute its expected value the way the code does?"
---
