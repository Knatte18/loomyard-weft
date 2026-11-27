```yaml
lens: fix-affected-set
outcome: reported
```

Lens fix-affected-set: I checked that each set-premised finding's fix covers every member of its set.

Findings left unchecked: none. F15 (the deletion set, plus the helpers and references it reaches) and F16 (the six fields, and the card 14 Phases and re-read sites) were both rerun.

**LOW: card 15, "Whole tests and helpers deleted", `TestNew_HappyPath` (`fabric_test.go`) entry.**
The file header of `internal/fabricengine/fabric_test.go` names the deleted test and the constructor it covered, and no card edits that header.
- `fabric_test.go:6` reads "TestNew_HappyPath tests the newPaired constructor at tier 0, spawning no git and using no hub fixtures."
- `fabric_test.go:1` reads "holds the unit tests for the Fabric handle, its sync options, and ScopedPathspec." Card 2 already moves `TestScopedPathspec` out of this file.
- After card 15 the header describes tests the file no longer holds.
- Card 15's Edit list has no entry for the header, though card 3 does name file-header edits for `junctionnames_test.go`.

**F15 set: no gap.** Every helper the deleted spans call still has another user:
- `setupCleanNonFastForward`: `mergein_recovery_integration_test.go:446`, `mergein_integration_test.go:173`
- `writeHandOverMark`: `takeoverrun_integration_test.go:90`, `rejectedtakeover_integration_test.go`
- `gitStatusPorcelain`: `bolt_integration_test.go:126`
- `newPlainWarpRepo`: other `commitweftat_test.go` tests and `pushdeferral_integration_test.go`
- `showAtTag`: `remove_records_integration_test.go:37`
- `findCleanupEntry`, `mustCreateOrphanWeftBranch`, `mustPushBranch`, `tagTargetAt`: many other files
- `dirtinessNA`, `dirtyScopeTracked`: production code
- `NewCommitFixtureForTest`, `ParseWarpSHATrailerForTest`, `CommitMessageAtForTest`, `WriteWarpFileForTest`: other `commit_integration_test.go` tests

`TestDropHandOverMark` calls the production `fabricengine.DropHandOverMark`, which keeps `TakeOverRun` as its caller, so the deletion leaves no test-only production code.

I also checked the `TestGate_CheckOrdering` deletion, because the test carries a `//testtiming:keep` ordering reason (`destroy_test.go:27`). The holder rows `WithinContainerPassesContainment` (`destroy_test.go:130-144`) and `AbsoluteOutsideContainer` (`destroy_test.go:106`) send the same two-check requests and assert the earlier check. The ordering stays held.

The deleted names appear elsewhere only in their own files, in their doc comments, and in `docs/benchmarks/test-redundancy.md` and `docs/research/.../internal_fabricengine.json`. Those are measurement reports and need no edit.

**F16 set: no gap.**
- Card 15 gives each of the six fields its error's `Store`, and the store order for the two multi-store fields.
- Card 14 covers both read sites: the phase request, where Phases is appended on success, and the re-read, which records the phase it ran under.
- The `TestRunRule` transport row now asserts Phases.

Tool calls beyond context:
1. Grepped helper identifiers in the deleted spans: my context lacked the deleted tests' bodies.
2. Counted the files using each helper and listed calls per deleted span: context lacked the helpers' users.
3. Located every use of each helper: needed to tell definition sites from other users.
4. Grepped the repository for references to the deleted test names: context lacked references outside the plan.
5. Read the `fabric_test.go` header, `TestGate_CheckOrdering`, and the gate containment rows: needed to confirm the header's staleness and the ordering holder.
