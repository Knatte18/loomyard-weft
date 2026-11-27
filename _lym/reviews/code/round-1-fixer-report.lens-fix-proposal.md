```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the review's proposals that were carried out as proposed (F1, F2, F3, F6, F7).

Proposals checked: F1, F2 and F6 have no finding; F7 and F3 each have one.

1. **MEDIUM**: `internal/webstercli/verbs_test.go` header, the line F7 added: "The way-forward tests reach fabricSync through a failing opener instead."
   - **Summary:** the sentence is false for some of the file's way-forward tests.
   - **Evidence:** `TestBracketVerbs_NoRunInProgressWayForward` (line 1969) builds its fixture with `newSkipGitVerbsFixture`.
   - **Evidence:** `TestFabricSyncWayForward_NextSyncCommitsSavedState` (line 1848) uses a failing opener and then a working `fabricengine.Open` over a real hub, with zero `SyncOptions`.
   - **Scope of the claim:** it holds only for the `*FabricSync…WayForward` verb tests that call `newVerbsFixture` and set `openFabric = failingFabricOpen` (lines 1880, 1906, 1938). Naming them "the fabric-sync failure tests" would be accurate.

2. **LOW**: `_lym/landing/summary.md`, the replacement sentence from F3: "After run 1 (5.4 s) is below both before runs and after run 2 (8.2 s) lies between them, so the gain is within the load swing."
   - **Summary:** the figures are right, but the line is 126 characters, over the 120-character limit for prose lines.
   - **Evidence:** `awk` reports a length of 126. Splitting it at ", so" or before "and after run 2" fixes it.

Tool calls:
- `grep` of `verbs_test.go` for tests and fixture setup: my context held only its first 40 lines.
- `sed` and `awk` over `verbs_test.go` lines 1843-1878 and 240-260, and the length of the `summary.md` line: those test bodies were not in context, and the length had to be measured.
