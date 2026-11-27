```yaml
lens: row-validity
outcome: reported
```

Lens: row-validity. For each planned test row I checked whether it reaches its branch, whether it would fail if the code were wrong, and whether it repeats an existing test.

**Findings**

1. **BLOCKING** (Card 6: `TestMutations_MarshalJSON` and `TestMutationRecord_EmbedsAndMarshalsUnderMutationsKey` move to the layout "unchanged but for the package")
   - Both tests build their records through the engine recorder: `*NewMutations("")`, `m.Append(...)` and `m.Snapshot()` (`internal/fabricengine/mutation_test.go:160-206`, `:233-236`).
   - None of those exist in `internal/fabriclayout`. An internal layout test importing `fabricengine` is an import cycle, since the engine imports the layout.
   - Moved unchanged, the tests do not compile. The rows have to be rebuilt from `fabriclayout.MutationsOf([]fabriclayout.Mutation{...})` or a literal `Mutations{}`, and the card has to say so.

2. **MEDIUM** (Card 6: `fabriclayout#MutationsOf`)
   - The Intent says `MutationsOf` "returns a record holding a copy of entries", but no planned row asserts the copy.
   - A row that changes the input slice after the call and checks `Entries()` is unchanged would fail if the copy were dropped.
   - `TestMutations_EntriesAndSnapshotAreCopies` stays in the engine and reaches only `Snapshot`.

3. **MEDIUM** (Card 2: `TestClassifyPaths` "holds today's rows")
   - Today's row `routingNames_with_more_than_two_entries` spells `BoltDirName` (`internal/fabricengine/classify_test.go:104-107`).
   - At card 2 that constant is still in the engine (card 3 moves it), and the layout package cannot import the engine.
   - The row needs the literal `"_bolt"` or `lymdirs.BoltDirName`. As written, it does not compile.

4. **LOW** (Card 2: the partition assertion folded into every `TestClassifyPaths` row)
   - Each row already checks all three buckets for exact, ordered equality against literals that together partition the input (`classify_test.go:157-165`).
   - So the per-row reassembly cannot fail when those three checks pass. It repeats them.
   - `TestClassifyPaths_PartitionsInputWithNothingLostOrDuplicated` can be deleted as a duplicate with no successor, and the fold dropped.

5. **MEDIUM** (Card 15: deleting `TestList_RefStorage`)
   - Today's test calls `List` from the main worktree and from a subdirectory of the linked worktree, under both the files and reftable backends (`internal/fabricengine/worktreelist_test.go:177-190`).
   - The replacement `TestSnapshotReader_RefStorage` reads through `snapshotReader.read`/`listWorktrees` with a store directory. No row covers `List` resolving a subdirectory source.
   - That behaviour loses its only test. Either keep a subdirectory row (the reader given a subdirectory, or `List` itself), or keep `TestList_RefStorage`'s subdirectory half.

6. **MEDIUM** (Card 15: the `TestSnapshotReader_RefStorage` files row, "read with `PATH` emptied, so the reader starts no git process")
   - The row names no fields.
   - `FieldOriginRefs` goes through `lsRemoteHeads` (`git ls-remote`), and `FieldRecordsStore` goes through `originNamespaceAt`, which reads git config.
   - With `PATH` empty, a row reading every field fails for the wrong reason. The card must limit the row to `FieldWorktrees` and `FieldLocalRefs`.
   - It must also state that `FieldAnchors`' `committedMarker` is out of the row, or that it runs in process.

7. **MEDIUM** (Card 15: `readLocalRefs`' fallback peels annotated tags)
   - Neither tier1 test plants an annotated tag. `TestSnapshotReader_ReadsAHub` asserts branches and tips only.
   - So the `for-each-ref` peel branch, and its agreement with the in-process `TagRefs` value, are reached by no row.
   - An annotated tag in the reftable row and in the files row, both wanting the commit SHA as a literal, would reach the branch and fail if the peel were wrong.

8. **MEDIUM** (Card 15: reader branches no row reaches)
   - `TestSnapshotReader_ReadsAHub` asserts "no pending teardown", which a reader that never reads `FieldPendingTeardowns` also passes.
   - The read-error wrapping ("wrapped with the field and the store it read"), the `ErrRecordsLayout` return and "a field no store answers is left empty" have no row.
   - Fix: plant a pending-teardown record in the hub, and add one row whose records store's layout is unreadable, wanting the wrapped `ErrRecordsLayout`.

9. **LOW** (Card 14: `TestOutcomeClassOf` transport row, "a `*gitexec.GitError` whose stderr `IsTransportFailure` reads as transport")
   - `IsTransportFailure` returns false unless `Args` names a remote subcommand (`internal/gitexec/transport.go:81`).
   - A row that sets only stderr falls to `OutcomeFailed`. The card must state that the row's `Args` begin with `push` or `fetch`.

10. **LOW** (Card 14: `runRule`, a nil seam meaning that is not stated)
    - When `start` is an action list, `Snapshot.Phases` is empty, so "re-reads the `Reread` fields under the latest phase" names no phase.
    - An answer carrying more than one of a refusal, a phase request and actions has no stated meaning, though `Answer` is documented "exactly one of".
    - The card should give both meanings: the zero `Phase`, and which part wins or that it is a wiring fault. No row is needed for a wiring guard.

Tool calls: 1. I read `mutation_test.go:148-255`, the scankit test list, the testkit enforcement test names, `TestList_RefStorage`, `IsTransportFailure`, and the `AnchorStopError` and `destructiveRefusal` declarations. My context held the plan but not those test bodies or helper rules.
