---
round: 2
focus:
  - "Check whether any function that only tests call still sits in a production file under the allowlist in cmd/lym/testonlycode_linux_test.go, although it reads no package state and writes no record, for example internal/shedcheck (Check, its helpers, Finding.String) and internal/shedbuild/check.go (Check). Settle whether PATTERN-test-only-code lets each allowlisted entry stay."
  - "Check that internal/fabricengine/replay.go calls ownedBoardWorktree (destroy.go) and that a _test.go file in package fabricengine pins its refusal of a board-named worktree no hub weft registers, via checkPathRequest and pathOwnershipBoardWorktree. Settle whether a deleted test left that live behaviour uncovered."
  - "Check internal/fabricengine/export_test.go for functions with no production caller whose behaviour is asserted by tests as a subject, for example PushAnchored and the tests driving it in pushanchored_integration_test.go and pushclasses_integration_test.go. Settle whether each is a reference or oracle, or a test-file copy of deleted production code."
  - "Check that no spec, doc or comment in the diff's range names a symbol the diff deletes or moves into a _test.go file, for example shedbuild.Load in contracts/specs/shed-recipe-spec.md, snapshotWarpSHA in internal/fabricengine/doc.go, AppendDecision in internal/discussionparser/append.go, planglyph.ValidateFormat in docs/overview.md, and boardcli.RunCLI in docs/benchmarks/board-performance.md."
---

Round 2 reviews the fixes to the gating findings and the stale-reference sweep fresh.
