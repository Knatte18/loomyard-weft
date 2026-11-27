```yaml
lens: fix-affected-set
outcome: reported
```

Task: re-run each set-based finding's search to check whether the fix covers every member.

no findings

Findings checked: F1, F2, F11 and F13 (each search rerun), F3/F10 (the set of unreached functions checked against what the cards create).
Findings left unchecked: none of the set-based ones. F4–F9, F12 and F14 are not set-based and are outside this lens.

- F1: the only whole-token `_bolt` spellings left in the cards are prose. They sit at `03-layout-hub-paths.md:86`, `:106` and `:110`, and plan markdown lies outside `TestEnforcement_BoltDirNameDeclaredOnce`'s walk roots, so no Go want spells it.
- F2: the other tier0 engine tests that call the moved hub path functions use the paths only as fixtures. Those are `failedpush_test.go:283`, `spawn_linux_test.go:44-75`, `remoteteardown_test.go:247-285`, `teardownlock_test.go:50`, `coalesce_test.go:129` and `moverecord_helper_test.go`. None pins a join that `TestHubPaths` now holds.
- F11: the tier0 engine tests of moved pure names come down to `TestCommitResult_Committed` (card 10 moves it), `TestOrigin_JSONRoundTrip` (card 11 moves it), `TestScopedPathspec` (card 2), `TestInWorktreeStaging` (card 3) and the archive-tag tests (card 4). `TestSeedCommitSignature` was already rejected in the review. The rest exercise the engine's recorder, gate or teardown code, which stays.
- F13: across `pattern`, `docs`, `internal`, `cmd`, `contracts` and `plugins`, the only loose spellings of a moved name's home are `pattern/PATTERN-single-repo-hub.md:11-12`, which card 4 now covers.
- F3/F10: the unreached set is runRule, outcomeClassOf, fieldReadError.Error, fieldReadError.Unwrap, snapshotReader.read, readLocalRefs and Refusal.Error. It matches every new production function no main reaches. The other new layout functions have engine callers.

Tool calls:
1. grep of the plan cards for whole-token `_bolt` (F1): my context did not hold the edited cards as one search.
2. A loop over tier0 engine test files for calls to the moved path functions (F2): my context held only the files the fixer read.
3. A loop over tier0 engine test files for moved pure methods and JSON of moved types (F11): same reason.
4. grep of `pattern`, `docs`, `internal`, `cmd` and `contracts` for "fabric's `&lt;moved name&gt;`" prose (F13): my context held only the sites the review named.
5. grep for "fabricengine's / the engine's `&lt;moved name&gt;`" prose (F13): same reason, for a second form of the spelling.
6. A loop over tier0 engine tests naming other moved pure names (F11): to cover the moved names the review did not list.
