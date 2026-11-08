# Cut and fold fabricengine tier 1 tests the coverage study found redundant

`fabric-test-cuts` · Task · enhancement, fabric, testkit · running · Darn-Write

## Seen

The coverage study `docs/research/speed/fabric-test-redundancy-2026-11-08/report.md` found that 370 of the 590 `internal/fabricengine` tier 1 tests cover no block another test does not, and that 267 unmarked ones can go together without uncovering a block (`greedy.tsv`).
Block coverage does not see assertions, and the report names several contained tests that guard data and must stay.

## Change (operator, 2026-11-08: fix it)

Cut and fold `internal/fabricengine` tier 1 tests from the report, following its "For the later task" section:

- Start from `greedy.tsv`, slowest first, and confirm each test against the module-wide check `go run ./cmd/testtiming -pkg ./internal/fabricengine -redundancy` before dropping it.
- Before dropping a test, read it and keep it when it asserts behaviour coverage cannot see: archive tags and their order, leased or other deletes on origin, user content never claimed or removed, mutation records and their order, machine-local files kept out of the weft tree.
  Mark a kept contained test `//testtiming:keep` with its reason.
- Make the folds the report names: the two `TestCleanup_*` tests onto one hub;
  `TestVerbCases_ArrangeSetsItsPreconditionField` into the shared check it supports, made to fail on an empty field;
  `TestStates`' vacuity check into the fixtures' own apply path;
  the anchor loops of `TestWireJunctions_ScratchLink`, `TestStates` and `TestCommitWeft_MachineLocalArtifactsNeverEnterWeftTreeAtAnyDepth` cut to the rows a nested anchor needs.
- Where a test asserts pure logic through git (`TestList`, `TestSourceDirtyPaths`, `TestUnwire_LeavesSiblingWorktreeUndirtied`), move the assertion to a tier 0 test when the parser or table can take its input without git; keep the git test otherwise.
- Drop a test whose rarest block count is 2 only with its partner staying.

## How the writer works (operator, 2026-11-08)

The work is too large for the darn writer alone: it must delegate to subagents through the Agent tool.
Split the tests into batches by source file, one subagent per batch, never two subagents on the same file;
each reads its tests, decides drop, keep or fold under the rules above, and makes the edits.
The writer reads their reports, settles anything they flag and runs the gate.

Measure the package's tier 1 time in a gate pass before and after, once each, and state both in the PR.
No production code changes beyond what a fold or a tier 0 move needs.
