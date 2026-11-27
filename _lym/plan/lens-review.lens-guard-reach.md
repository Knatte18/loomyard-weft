```yaml
lens: guard-reach
outcome: reported
```

Task: guard-reach review of the fabric core foundation plan (00-overview.md and cards 1–15).

I found no BLOCKING finding. Four MEDIUM and five LOW defects follow, each with its guard or branch and its evidence.

1. **MEDIUM — Card 15, `FieldLocalRefs`' non-`files` fallback: the fallback never runs for the records and Bolt stores.**
   - The card makes the reader learn a store's ref storage "from the same `dotgit.ErrRefStorage` probe of that store".
   - The only probe it names is `listWorktrees`, and `FieldWorktrees` lists only "the code store's worktrees".
   - So a records or Bolt store whose storage is not `files` has no stated probe. It goes to `gitrepo`'s `BranchRefs`/`TagRefs`, whose go-git open fails (D2). The read then fails where the fallback should have taken over.
   - The same happens when `FieldLocalRefs` is asked for without `FieldWorktrees`.
   - `TestSnapshotReader_RefStorage` builds only one reftable store, so it never reaches this.
   - Evidence: card 15's Fields list, the `FieldWorktrees` and `FieldLocalRefs` bullets; `internal/gitrepo/historyreads.go:35`, `:91` (both are `readGoGit`).

2. **MEDIUM — Card 15, `TestSnapshotReader_RefStorage`'s empty-`PATH` row: the fields it reads are unstated, and two of them must spawn git.**
   - The row says "a files store is read with `PATH` emptied, so the reader starts no git process".
   - `FieldOriginRefs` goes through `lsRemoteHeads`, which runs `ls-remote`.
   - `FieldAnchors` goes through `committedMarker`, which calls `showFile` (`internal/fabricengine/anchormarker.go:85-86`).
   - If the row reads every field, as `TestSnapshotReader_ReadsAHub` does, it fails on a correct reader.
   - The row needs to name `FieldWorktrees` and `FieldLocalRefs` alone.

3. **MEDIUM — Cards 2 and 12, `TestLeafInvariant_NoIO`: the I/O guard is bypassable.**
   - `bannedImports` reports an import only when its "path equals a `banned` entry" (card 1).
   - The ban list is `os`, `os/exec`, `syscall`, `net`. So `io/ioutil`, `net/http`, `os/user` and `os/signal` all pass.
   - `path/filepath`'s `Abs`, `EvalSymlinks`, `Glob` and `Walk` touch the filesystem with no import of `os` at all.
   - The engine's own `refDetail` already uses `filepath.Abs` (`internal/fabricengine/mutation.go:177-178`), so this is a real reach for a file moved into the layout.
   - The rule this guard is meant to hold, D1's "does no I/O", can therefore break with the guard staying green.

4. **MEDIUM — Card 14, `runRule`'s re-read after a failed action: the outcome can be lost before the rule sees it.**
   - The card ends the list on the first non-done outcome, then "once the list ends, `read` re-reads the answer's `Reread` fields", and "a `read` error is returned as it is".
   - When a push ends with `OutcomeTransportFailure` and `Reread` names `FieldOriginRefs`, the re-read's `ls-remote` fails on the same unreachable origin.
   - The loop then returns a raw read error, and the rule never sees the transport outcome. D3 says "its outcome returns to the core", and the rule alone decides when the loop stops.
   - No `TestRunRule` row combines a failed action with a failing re-read.

5. **LOW — Card 14, re-read "under the latest phase" when no phase has been read yet.**
   - `start` may itself be an action list. `Snapshot.Phases` is then empty when the list ends, and the card does not say which phase `read` gets.
   - No row covers this.

6. **LOW — Card 15, a `FieldRecordsStore` re-read under the lock cannot see a change.**
   - `originNamespaceAt` caches per directory in the process-wide `namespaceAnswers` (`internal/fabricengine/recordslayout.go:68-70`).
   - A D3 re-read of this field after an action, or a later phase, returns the first answer.

7. **LOW — Card 15, `FieldAnchors` without `FieldWorktrees`.**
   - The card reads "each records worktree's committed marker" but names no source of the records worktrees when `Snapshot.Worktrees` has not been read yet.
   - The field then comes back silently empty.
   - Separately, the "a worktree with none has no entry" branch is reached by no row: a hubforge pair carries a marker.

8. **LOW — Card 15, the two ref-read paths can disagree on tags.**
   - `TagRefs` peels an annotated tag and skips a tag that names no commit (`internal/gitrepo/historyreads.go:33-34`).
   - The `git for-each-ref` path is stated to peel, but not to skip such a tag.
   - The two paths can then return different ref sets for one store.

9. **LOW — Card 6, `TestMutationRecord_FabricengineProductionSource`'s widened first-field check lacks a refusal row.**
   - The guard compares the first field's text exactly (`cmd/lym/destructiveguard_test.go:290`). Admitting `fabriclayout.MutationRecord` is right.
   - The card states no row where a listed type embeds some other first field, so the widened comparison's refusal goes unreached by any planned row.

Tool calls:
- Read `cmd/lym/destructiveguard_test.go`'s `TestMutationRecord_FabricengineProductionSource` and its table names. My context lacked how the guard reads result types: listed files, an exact first-field match.
- Read `gitrepo`'s `BranchRefs`/`TagRefs`, `committedMarker` and `originNamespaceAt` bodies. My context lacked whether they spawn git, peel tags or cache.
