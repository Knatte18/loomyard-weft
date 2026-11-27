```yaml
lens: guard-reach
outcome: reported
```

**Guard-reach list: each guard or branch the cards add, and whether it is reachable**

- Card 1, `bannedImports` exact ban, `/...` ban, the skips of `_test.go` files and subdirectories, and its errors for a missing directory and an unparsable file: reachable.
- Card 1, the `AssertNoImports` rows run on a `recordingTB`: **not reachable as specified** (finding 2).
- Cards 2 and 12, `TestLeafInvariant_AllowlistOnly`: reachable. Its NoStale check is met because the package imports `lymdirs` (card 2), then `weftname` (card 4) and `fabriclayout` (card 12).
- Cards 2 and 12, `TestLeafInvariant_NoIO`: reachable. `os/exec` is caught through `os/...`.
- Card 2, the append row of `TestStructuralDirs`: reachable.
- Card 6, the true and false rows of `isMutationRecordEmbed`: reachable.
- Card 12, `Refusal.Error` with and without a way forward: reachable.
- Card 12, an answer with more than one part (wiring fault): exempt, a fail-loud wiring guard.
- Card 13, the setter-after-`Build` row and `TestEnforcement_FabrickitLeafOnly`: reachable.
- Card 14, the `runRule` branches for refusal, empty-reason refusal, phase request, action list, start-answer re-read, failed action, wait, transport read failure, other read error and empty answer: reachable through injected rule, read and runners.
- Card 14, the transport read failure from the real reader: reachable. `lsRemoteHeads` wraps the `GitError` of `ls-remote` with `%w` (`internal/fabricengine/originreconcile.go:546-549`), and `IsTransportFailure` takes remote subcommands.
- Card 14, the multi-part answer and missing-runner wiring faults: exempt.
- Card 14, the `outcomeClassOf` rows: reachable in the internal package. Each error is built in process, and `destructiveRefusal` is unexported but visible there.
- Card 15, the `files` rows with `PATH` emptied: reachable. `List` uses `dotgit` in process, and `BranchRefs` and `TagRefs` go through go-git (`internal/gitrepo/historyreads.go:35,91`).
- Card 15, the reftable fallback rows for worktrees and local refs: reachable at git ≥2.45.
- Card 15, the `FieldAnchors` row with the marker removed: reachable, since `committedMarker` reports found false for an absent file (`internal/fabricengine/anchormarker.go:85-95`).
- Card 15, the row with only the code store named and every other field empty: reachable.
- Card 15, the `ErrRecordsLayout` row: **not reachable as specified** (finding 1).
- Card 15, the ref-storage fallback: **the site recurs unguarded** on `FieldAnchors` and `FieldRecordsStore` (finding 3).

**Findings**

1. MEDIUM, consistency. Location: `_lym/plan/15-snapshot-reader.md`, the fourth `TestSnapshotReader_ReadsAHub` row.
   - Summary: the row unsets the records store's fetch refspec and expects `FieldRecordsStore` to fail with `ErrRecordsLayout`. As written it gets the cached good layout instead: `originNamespaceAt` caches each store's answer for the process, and the hub build already read and cached it. The card never calls `fabricengine.ForgetOriginNamespaceForTest(h.PrimeRecords())` after breaking the refspec, nor lists it under Uses.
   - Evidence: the cache at `internal/fabricengine/recordslayout.go:67-70`; the existing test of the same state calls `ForgetOriginNamespaceForTest` right after `config --unset-all remote.origin.fetch` at `internal/fabricengine/onerepository_lifecycle_integration_test.go:49-50`; the helper is at `internal/fabricengine/export_test.go:659-662`.
   - Fix: the row forgets the store's cached layout after rewriting its config, and Uses names `internal/fabricengine#ForgetOriginNamespaceForTest`.

2. MEDIUM, consistency. Location: `_lym/plan/01-scankit-banned-imports.md`, the `AssertNoImports` rows on `recordingTB`.
   - Summary: `recordingTB` embeds `testing.TB` and records only `Errorf`, `Fatalf` and the failed state, so the embedded interface stays nil.
   - `AssertNoImports` calls `t.Helper()`, as `assertImports` does, and `RequireFloor` calls `t.Helper()` and `t.Error`. Each call through the nil embedded TB panics, so the "empty directory records the floor failure" row cannot reach its state.
   - Also, a recorded `Fatalf` does not stop execution as a real one does. The missing-directory row therefore runs on into `RequireFloor` and records a second message, besides the one fatal message the card expects.
   - Evidence: `RequireFloor` at `internal/testkit/scankit/scankit.go:281-286`, which calls `t.Helper()` then `t.Error(err)`; `assertImports` at `internal/testkit/scankit/scankit.go:358-368`.
   - Fix: `recordingTB` overrides `Helper`, `Error`, `Errorf` and `Fatalf`, and `AssertNoImports` returns after the `Fatalf` on the scan error, so the missing-directory row records exactly one fatal message.

3. LOW, design. Location: `_lym/plan/15-snapshot-reader.md`, `FieldAnchors` and `FieldRecordsStore`.
   - Summary: the card guards the non-`files` ref-storage case for `FieldWorktrees` and `FieldLocalRefs` alone. On a reftable records store, `FieldAnchors` reads through `committedMarker`, which calls `showFile` and so `gitrepo` and go-git, and go-git refuses the open. `FieldRecordsStore` reads through `originNamespaceAt`, which also uses `gitrepo`.
   - Those errors reach `outcomeClassOf` as `failed` and return raw, with no git fallback and no row that pins it. D2 asks for the fallback for worktrees and refs only, and today's reads behave the same way, so this is no regression. The card's doc text on the per-store probe and fallback should still say that those two fields have no fallback.
   - Evidence: `committedMarker` at `internal/fabricengine/anchormarker.go:85-86`; `showFile` reading through `gitrepo.New` at `internal/fabricengine/configchanges.go:116-124`; the `gitrepo` read in `originNamespaceAt` at `internal/fabricengine/recordslayout.go:76`.

**Tool calls**

- Read `RequireFloor`, the existing scankit fake TBs and the `gitrepo` `TagRefs`/`BranchRefs` bodies: needed to know what a `recordingTB` must implement and whether the reader's `files` path spawns git, which my context did not show.
- Read `CloneAndWire` and `onerepository_lifecycle_integration_test.go:40-55`: needed to confirm that a one-repository hub build caches the namespace and that the existing test forgets it, which my context did not show.
