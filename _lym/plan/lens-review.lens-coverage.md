```yaml
lens: coverage
outcome: reported
```

Coverage lens: checked every decision, acceptance criterion and Intent branch of the plan for a card that carries it and a planned row that reaches it.

**Findings**

1. MEDIUM — D7 / acceptance criterion 2, and the Shared Decision "what the layout takes". Several pure names that files outside fabric use are moved by no card, so their importers keep the `internal/fabricengine` import. The Shared Decision says the layout takes every such name and that "the cards list every moved name". Card 4 calls the trailer parsers engine code, but `TaskCommitOf` is a pure string function in trailer.go, which the exploration found pure as a whole.
   - `TaskCommitOf` is used by `internal/reportengine/landings.go` and `internal/webstercli/reset_test.go`.
   - `IsReadOnlyCommand` (pure string logic, refscanner.go) is used by `internal/webstercli/{run,recordbatch,recoverbatch,acceptaudit}.go`, `internal/loomcli/wiring.go` and four websterengine tests.
   - `FabricReaderVerbs` is used by `internal/fabriccli/roleguard.go` and its test.
   - `SeedCommitMessage` is used by `cmd/lym/stencilseed.go` and its integration test.
   - `SyncCommitMessage` is used by `internal/stencilcli/cli.go`.
   - `InWorktreeStaging` is used by `internal/beamcli/hubstop.go`, `internal/orchcli/live.go` and `internal/heddlecli/verbs.go`.
   - Either move them (cards 4 and 11) or record in the Shared Decision why each stays.

2. MEDIUM — card 6, `cmd/lym#TestMutationRecord_FabricengineProductionSource`. The changed branches have no reaching row: accepting `fabriclayout.MutationRecord` as a first field, and refusing it on a read-only type. The guard only scans the real tree (`cmd/lym/destructiveguard_test.go:290`, `firstField != "MutationRecord"`). Nothing would fail if the qualified spelling were wrongly accepted on a read-only type, or wrongly refused on a mutating one, as long as the tree happened to comply. The card plans no fixture or unit row for `structFirstFieldLine` or the comparison.

3. MEDIUM — card 15 branches with no reaching row. `TestSnapshotReader_ReadsAHub` builds one hubforge hub with one pair, and `TestSnapshotReader_RefStorage` covers only the files/reftable split. These branches are reached by neither:
   - `FieldAnchors` for a worktree with no committed marker ("has no entry").
   - `FieldRecordsStore` with `Present` false, and the `ErrRecordsLayout` return.
   - "a field no store or hub path answers is left empty".
   - "a read error is returned wrapped with the field and the store".
   - `FieldPendingTeardowns` with a non-empty record (only "no pending teardown" is asserted).
   - The reftable `for-each-ref` path's peeling of an annotated tag: the row does not say it creates one.

4. MEDIUM — D2 ("a verb that refuses early reads no more than it does today"; "no verb starts more git processes"), card 15. `read` "reads only the fields a phase names" and "leaves the rest alone", but no row reads a subset of fields and asserts that the other fields stay empty, or that no git starts. The `PATH`-emptied row covers only the files-store worktree and local-ref reads.

5. LOW — card 14, `runRule`. The `read` error row does not say whether it covers the re-read after an action list, which is a separate call site from the phase-request read. Also unstated: the `Reread` phase when no phase was read yet (`Snapshot.Phases` empty, start answer an action list). That is an exit with no defined operand and no row.

6. LOW — card 1, `AssertNoImports`. `TestBannedImports` drives only `bannedImports`. The wrapper's own branches have no row and are reached only by passing leaf tests: the error turned into a failure, the `RequireFloor` on an empty directory, and the `t.Errorf` naming the package and path. The Intent promises a scan of an empty directory "fails loudly".

7. LOW — card 2 `StructuralCommittedDirs` / `StructuralNeverCommittedDirs`, and card 6 `MutationsOf`. Each Intent states a copy guarantee: "a fresh" slice so no caller can change the sets, and "a copy of entries". No row mutates a returned slice and re-reads to show the guarantee holds.

8. LOW — acceptance criterion 4 / D6 ("each flow shape has one tier1 seam test that … runs core and executor, and asserts the end state"). Card 15 plans only the snapshot half and defers the core and executor half to part 2. This is acceptable since no flow moves in part 1 (D5, D9), but the overview does not record the deferral as a Shared Decision.

9. LOW — D2 operand ("local and origin refs of the code store and the records store"). Card 15's `FieldOriginRefs` reads origin heads only through `lsRemoteHeads`, so origin tags are never read. Part 3's archive-tag rules read origin tags, and the fields table does not name the gap.

Tool calls:
- Grepped `stringSlicesEqual` users and read `junctionnames_test.go:104-190`, to check what card 2's and card 3's test deletions drop; my context lacked those test bodies.
- Read `TestMutationRecord_FabricengineProductionSource` and grepped the importers of six unmoved pure names, to check card 6's guard rows and D7's completeness; my context lacked the guard body and the per-name importer files.
- Grepped `structuraldirs_test.go` for `stringSlicesEqual` callers, to confirm card 2 covers the only remaining user; my context lacked that file's members.
