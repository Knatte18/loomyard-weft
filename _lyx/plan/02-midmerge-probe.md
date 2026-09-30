# Card 2 — midmerge-probe

**Edit:**
- `internal/fabricengine#Fabric.foreignMergeStatePresent`
- `internal/fabricengine/doc.go`

**Create:**
- `internal/fabricengine/midmerge.go`
- `internal/fabricengine#MidMergeKind`
- `internal/fabricengine#MidMergeNone`
- `internal/fabricengine#MidMergeParked`
- `internal/fabricengine#MidMergeForeign`
- `internal/fabricengine#MidMergeState`
- `internal/fabricengine#MidMerge`
- `internal/fabricengine/midmerge_integration_test.go`

**Uses:**
- `internal/fabricengine#Open`
- `internal/fabricengine#Fabric.mergeRecordExists`
- `internal/fabricengine#resolveMergeGeometry`
- `internal/fabricengine#unifyConflictPaths`
- `internal/fabricengine#MergeStateActive`
- `internal/fabricengine#Fabric.MergeIn`
- `internal/fabricengine_test#newMergePairFixture`
- `internal/fabricengine_test#setupConflictingDivergence`
- `internal/fabricengine_test#setupResolvedConflictedMergeIn`
- `internal/fabricengine_test#gitMergeAllowConflict`
- `internal/fabricengine_test#gitMergeSquashAllowConflict`
- `internal/fabricengine_test#gitMergeHeadRaw`
- `internal/fabricengine_test#gitUnmergedPaths`
- `internal/fabricengine_test#foreignShape`

**Intent:**
Add the read-only, `l`-in, vocabulary-neutral mid-merge probe `lyx loom start` will consult, in a new file `internal/fabricengine/midmerge.go`.

The result type `MidMergeState` carries `Kind MidMergeKind` and `Conflicts []string`.
`MidMergeKind` has three values, declared as one `iota` block:
`MidMergeNone` (no merge state anywhere on the pair), `MidMergeParked` (the fabric merge-state record exists), and `MidMergeForeign` (no record, but a live `MERGE_HEAD` or unmerged index entries on either side of the pair).
Neither the type, its fields, its constants nor their doc comments name the two sides, since the result is read from `internal/loomcli`, outside the Fabric Vocabulary Invariant's owner set.
The type embeds no `MutationRecord` (Mutation Record Invariant: a read-only probe must not).

`MidMerge(l)` behaves as follows:

- Open the pair with `Open(l)`.
  An open failure is returned as a wrapped error, never as `MidMergeNone`.
- Read the record's existence through `Fabric.mergeRecordExists`.
  Its error, which includes a record location that cannot be resolved (the weft gitdir), is returned wrapped — deliberately unlike `mergeBlocksMutation`, which swallows an unopenable pair to false: an undetermined answer is not evidence the tree is clean.
- Read the four git-level probes `foreignMergeStatePresent` combines (`MERGE_HEAD` and conflicted files on each side), all four unconditionally.
  Extract them into one unexported reader in `mergestate.go`, directly above `foreignMergeStatePresent`, returning the four raw readings (two `MERGE_HEAD` booleans, two conflicted-path lists), and rewrite `foreignMergeStatePresent` as a thin combination over that reader, so the four calls exist once and `MidMerge` in `midmerge.go` calls the same reader.
  Keeping the reader in `mergestate.go` keeps every foreign-state probe in the file whose header says it holds them.
  `foreignMergeStatePresent`'s observable answer and its error wrapping stay byte-identical;
  its existing doc comment moves with the four calls where it describes them.
- Classify: record present → `MidMergeParked`; else any `MERGE_HEAD` or any conflicted path → `MidMergeForeign`; else `MidMergeNone`.
  A record wins over foreign-looking git state, since a parked fabric merge leaves `MERGE_HEAD` and unmerged entries of its own.
- Map the two conflicted lists onto the pair's one-repo form with `resolveMergeGeometry(l)` plus `unifyConflictPaths`, the same mapping a fabric merge verb's `conflicts` uses, so `lyx fabric merge-stage` accepts every listed path.
  Resolve the geometry only when at least one side has a conflicted path, so a clean pair's answer never depends on the config read.
  A geometry error, or an unmappable path (`unifyConflictPaths` reporting `unmappable`), is returned as an error naming the condition rather than a partial list.
  `Conflicts` is empty-never-nil.

The doc comment on `MidMerge` states why neither existing probe serves: `MergeStateActive` is weft-only and `Fabric.MergeInProgress` is record-only, and each misses the incident shape (a conflicted warp-side merge-in, or foreign warp-side state).

Update `internal/fabricengine/doc.go`'s one-repo-illusion section: add `MidMerge(l)` to the list of vocabulary-neutral, `l`-in entry points beside `PushAnchored`, `MergeStateActive` and `RequireDrivableWorktree` (the sentence currently counts "three further" — reword it so it does not carry a count), with one sentence saying what it answers and that `lyx loom start` consults it before putting a driver to work.
Semantic line breaks in every new or changed comment line.

Bundled test, `internal/fabricengine/midmerge_integration_test.go` (`//go:build integration`, package `fabricengine_test`, reusing the package's existing `TestMain` and fixtures), one row per state, each asserting the exact `Kind` and the exact `Conflicts` slice:

- Clean pair → `MidMergeNone`, empty `Conflicts`.
- Fabric-parked with conflicts: `setupConflictingDivergence` on the prime worktree, then `Fabric.MergeIn` → `MidMergeParked`, and `Conflicts` equal to the `MergeIn` result's own `Conflicts` (pins the one-repo form).
- Fabric-parked with none left: `setupResolvedConflictedMergeIn` (the conflict resolved and staged, the merge not concluded) → `MidMergeParked`, empty `Conflicts`.
- Foreign, each of the three shapes (`git merge` conflicted, `git merge` resolved but unconcluded, `git merge --squash` conflicted) on each side, shaped exactly as `TestMergeVerbs_ForeignMergeState_EverySideAndShapeRefuses` builds them (weft-side conflict path under `_lyx/`) → `MidMergeForeign`, with `Conflicts` equal to the one conflicted path for the two conflicted shapes and empty for the resolved-unconcluded shape.
- A location whose pair cannot be opened (a `lyxcwd.Location` over a `t.TempDir()` with no worktree behind it) → a non-nil error.

**ImpactSummary:** foreignMergeStatePresent has 6 call sites, all in fabricengine (MergeIn, Merge and recheckMergePreconditionsUnderLock in merge.go, mergeStateOrForeignErr for MergeContinue/MergeAbort, MergeStageResolved, Fabric.Commit), and each sees an unchanged answer and error wrapping through the extracted reader; doc.go is comment-only, and MidMerge has no caller until card 3.

**Commit:** 2: fabricengine: add the MidMerge probe for a pair's unfinished merge
