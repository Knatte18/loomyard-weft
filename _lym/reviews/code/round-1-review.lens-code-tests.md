```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, find the test row that would fail if it broke.

**Behaviour → pinning test**

`runRule` (`executor.go`):

| Behaviour | Pinned by |
|---|---|
| Start refusal, including an empty reason, is returned with no read or lock | `StartRefusalEndsAtOnce` |
| Phase request locks, reads and records the phase | `PhaseThenActionsThenRereadUnderThatPhase` |
| One lock per phase | `TransportReadFailuresAreRecorded…` |
| Releases run in reverse order | `LocksPrecedeTheirReadsAndReleaseInReverse` |
| Lock error means no read | `LockErrorReadsNothing` |
| A nil `lock` takes none; a nil `sleep` sleeps for real | none (F7) |
| A failed phase read does not enter `Phases` | `TransportReadFailures…` |
| A phase read twice successfully appears once | none (F4) |
| Action order, and a failure ends the list | `PhaseThenActions…`, `FailedActionSkipsTheRest…` |
| Wait sleeps its delay | `WaitSleepsItsDelay` |
| Re-read under the latest phase after a successful read | `PhaseThenActions…` |
| Re-read under no phase | `StartActionsRereadUnderNoPhase` |
| Re-read under a phase whose read failed | none (F2) |
| Joined transport failures on a phase read are recorded in order | `TransportReadFailures…` |
| Transport failure on a re-read is recorded and the loop continues | none (F2) |
| A joined error holding a non-transport part, or a `*fieldReadError` wrapping a non-transport error, is returned raw | none (F3) |
| Other error on a re-read keeps the outcomes | `OtherRereadErrorKeepsTheOutcomes` |
| The two wiring faults | none; card 14 says they take no row, so not raised |

Other engine code:

| Behaviour | Pinned by |
|---|---|
| `outcomeClassOf`, each class | `TestOutcomeClassOf` |
| `fieldReadError` `Error` and `Unwrap` | `RecordsStoreWhoseLayoutCannotBeRead` |
| Reader: worktrees, both backends | `TestSnapshotReader_RefStorage` |
| Reader: local refs and annotated peel, both backends | `RefStorage`, `ReadsAHub/EveryField` |
| Reader: a tag naming no commit is skipped | none (F6) |
| Reader: origin heads and `OriginRead` | `EveryField` |
| Reader: a failing store does not stop the walk; one joined error per store, in order | none (F1) |
| Reader: anchors, and an unmarked worktree has no entry | `EveryField` |
| Reader: a prunable records worktree is skipped | none (F6) |
| Reader: records store present with identity layout | `EveryField` |
| Reader: mapped layout fills `Heads` and `Tags` | none (F5) |
| Reader: an absent records directory gives `Present` false | none (F5) |
| Reader: layout error | `RecordsStoreWhoseLayoutCannotBeRead` |
| Reader: pending teardowns | `EveryField` |
| Reader: only the named fields are read | `LocalRefsOfTheCodeStoreAlone` |

Leaves, layout and guards:

| Behaviour | Pinned by |
|---|---|
| `bannedImports` / `AssertNoImports` | `TestBannedImports` |
| Layout and core import sets and I/O ban | `TestLeafInvariant_*` |
| Kit import set | `TestEnforcement_FabrickitLeafOnly` |
| Builder | `TestBuilder` |
| `Refusal.Error` | `TestRefusal_Error` |
| Moved hub path functions | `TestHubPaths`; every function in `hubpaths.go` with a string or list result has a row there or in `TestIsReservedHubName` / `TestInWorktreeStaging` |
| `isMutationRecordEmbed` | `TestIsMutationRecordEmbed` |

Deletions I spot-checked keep their assertion elsewhere: the warp-first trailer (`TestCommitWithTrailers/TrailersEndTheWeftMessageOnly` checks `Warp-SHA` equals `result.WarpSHA`) and the archived drive report (`TestRemove_CommitsPendingRecordsIntoArchiveTag`). I did not re-check the other card 15 deletions.

**Findings**

**F1. MEDIUM, design.** `internal/fabricengine/snapshotread.go:37-53,156`
- The multi-store partial fill has no test: a failing store does not stop the walk, the other stores are still filled, `errors.Join` holds one `*fieldReadError` per failing store in walk order with its `Store`, and a failed store's refs stay as they were.
- Card 15 states this behaviour, and it is what lets an unreachable code origin not hide the records origin.
- Replacing `continue` after `fail` with a return would pass every test. `ReadsAHub` has only an all-success read and a single-store failure, and `TestRunRule` fakes `read`.
- Evidence: `internal/fabricengine/snapshotread_integration_test.go:46-146`.

**F2. MEDIUM, design.** `internal/fabricengine/executor.go:156`
- No row gives a re-read a transport failure. Changing the line to `if err := read(...); err != nil { return snapshot, err }` passes every row, although an unreachable origin on a re-read would then abort the verb instead of reaching the rule.
- The `Phase` stamped on such a failure is also unpinned, including "latest phase requested, even if its read failed".
- Evidence: `internal/fabricengine/executor_test.go:146-173`. The only re-read error row uses `errors.New("disk error")`.

**F3. MEDIUM, design.** `internal/fabricengine/executor.go:74-89`
- No row has a joined error mixing a transport failure with another failure, or a `*fieldReadError` wrapping a non-transport error.
- Recording the transport parts and swallowing the rest (an "any" instead of "all" rule), or dropping the `outcomeClassOf` condition, passes every row. A real read error would then be silently turned into an unreachable origin.
- Evidence: `internal/fabricengine/executor_test.go:146-173`.

**F4. LOW, design.** `internal/fabricengine/executor.go:129`
- No row reads one phase successfully twice, so removing `!slices.Contains(snapshot.Phases, latest)` passes.
- The one repeated-phase row fails its first read.
- Evidence: `internal/fabricengine/executor_test.go:146-161`.

**F5. MEDIUM, design.** `internal/fabricengine/snapshotread.go:129-141`
- `FieldRecordsStore` on a one-repository hub with a readable layout is untested. `EveryField` wants empty `Heads` and `Tags`, so dropping `Heads: namespace.Heads, Tags: namespace.Tags` passes every test.
- The absent-records-directory case (`Present` false) has no row either.
- Evidence: `internal/fabricengine/snapshotread_integration_test.go:103-105,132-145`.

**F6. LOW, design.** `internal/fabricengine/snapshotread.go:105-107,171-179`
- Two skip behaviours are untested.
- A tag naming a tree or blob is left out. The `for-each-ref` branch has its own hand-written filter, and no repository in `RefStorage` holds such a tag (`snapshotread_integration_test.go:190-191`).
- A prunable records worktree is skipped in the anchors read. No fixture has one, and without the skip the read would run git in a missing directory.

**F7. NIT, design.** `internal/fabricengine/executor.go:63-65,117`
- No row passes a nil `lock`, which card 14 says "takes none". Every row passes a non-nil one (`executor_test.go:219-237`).
- The nil-`sleep` default is likewise unrun.

Tool calls:
1. `hubpaths_test.go` and the function list of `hubpaths.go`: the manifest had no layout tests.
2. `git show f4fd75355` removed lines for four test files, plus a grep of `trailercommit_integration_test.go` (it matched nothing): the card 15 deletion hunks were not in context.
3. `TestCommitWithTrailers` and the `Warp-SHA` assertions: the named holder of the deleted warp-first test was not in context.
4. `TestRemove_CommitsPendingRecordsIntoArchiveTag`: the named holder of the deleted drive-report test was not in context.
