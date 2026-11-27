# Card 15 — snapshot-reader

**Create:**
- `plan:internal/fabricengine#snapshotReader` -> `type snapshotReader struct`
- `plan:internal/fabricengine#snapshotReader.read` -> `func (r snapshotReader) read(phase fabriccore.Phase, fields []fabriccore.Field, snapshot *fabriccore.Snapshot) error`
- `plan:internal/fabricengine#readLocalRefs` -> `func readLocalRefs(dir string) (fabriccore.Refs, error)`
- `plan:internal/fabricengine#ReadSnapshotForTest` -> `func ReadSnapshotForTest(hub string, stores map[fabriccore.Store]string, phase fabriccore.Phase, fields []fabriccore.Field) (fabriccore.Snapshot, error)`
- `plan:internal/fabricengine#WritePendingTeardownForTest` -> `func WritePendingTeardownForTest(hub string, record fabriclayout.PendingTeardown) error`
- `plan:internal/fabricengine_test#TestSnapshotReader_ReadsAHub` -> `func TestSnapshotReader_ReadsAHub(t *testing.T)`
- `plan:internal/fabricengine_test#TestSnapshotReader_RefStorage` -> `func TestSnapshotReader_RefStorage(t *testing.T)`

**Edit:**
- `cmd/lym#testOnlyCodeAllowed`

**Delete:**
- `internal/fabricengine_test#TestList_RefStorage`

**Extends:**
- `internal/fabricengine/doc.go`

**Uses:**
- `plan:internal/fabricengine#snapshotRead`
- `internal/fabricengine#List`
- `internal/fabricengine#lsRemoteHeads`
- `internal/fabricengine#originNamespaceAt`
- `internal/fabricengine#committedMarker`
- `internal/fabricengine#PendingTeardowns`
- `internal/fabricengine#writePendingTeardown`
- `internal/fabricengine_test#TestList_ParsesPrunable`

**Intent:** Add the phased snapshot reader (D2) in `internal/fabricengine/snapshotread.go`: the `snapshotRead` the executor loop calls, reading only the fields a phase names.
`snapshotReader` holds the hub path and each store's repository directory; `read` fills the named fields of `snapshot` and leaves the rest alone, so a verb that refuses early reads no more than it does today.

Fields and their reads, all through `gitexec`, `gitrepo` or in-process parsers (`PATTERN-gitrepo-client-boundary`):
- `FieldWorktrees` lists the code store's worktrees through `List`, which reads `dotgit.ListWorktrees` in process and falls back to `git worktree list --porcelain` on `dotgit.ErrRefStorage`, as it does today.
- `FieldLocalRefs` reads each store's branches and tags through `readLocalRefs`, keyed `refs/heads/<name>` and `refs/tags/<name>`.
  `readLocalRefs` probes its own store with `dotgit.ListWorktrees`, in process, so every store, the records and Bolt stores included, learns its ref storage with no git process.
  A `files` store is read in process through `gitrepo`'s `BranchRefs` and `TagRefs`.
  A store whose ref storage is not `files` is read through `git for-each-ref`, each annotated tag peeled to its commit and a tag naming no commit skipped, as `TagRefs` does, since go-git refuses to open such a store and so never yields a wrong ref (D2).
- `FieldOriginRefs` reads each store's origin heads through `lsRemoteHeads` with zero options, which maps a records store's names through its layout, and sets `OriginRead`.
- `FieldAnchors` lists the records store's worktrees through `List` itself and reads each one's committed marker at `HEAD` through `committedMarker`, so it needs no other field; a worktree with no committed marker has no entry.
- `FieldRecordsStore` sets `Present` when the reader holds a records store directory that exists, and `Heads` and `Tags` from `originNamespaceAt`.
  The layout is read through `originNamespaceAt`'s per-process cache, as today's verbs read it, so a later re-read of this field in one process answers the first read; the doc says so, and a rule reads it in its first phase.
- `FieldPendingTeardowns` reads `PendingTeardowns(hub)`.
A field no store or hub path answers is left empty.
Every read error is returned wrapped with `%w`, naming the field and the store it read, so `errors.Is` still finds `ErrRecordsLayout` and its message keeps the refspecs to restore.

`snapshotReader.read` and `readLocalRefs` have no production caller until part 2, so `cmd/lym#testOnlyCodeAllowed` admits `internal/fabricengine.snapshotReader.read` and `internal/fabricengine.readLocalRefs`, each reason naming this card's tests and part 2 as the first caller.
In `export_test.go`, `ReadSnapshotForTest` builds a reader and reads one phase, and `WritePendingTeardownForTest` writes a pending-teardown record through `writePendingTeardown`, for the external test package, which builds its hubs through `internal/hubforge`.

Tests, tier1:
- `TestSnapshotReader_ReadsAHub` (table) is the reader's seam test (D6), the snapshot half of the first flow's; part 2 adds the core and executor half. Rows:
  - a hub with one pair, an annotated tag on the prime's records branch and a pending-teardown record planted through `WritePendingTeardownForTest`, every field read:
    it wants the prime's and the pair's branches and their tips, the annotated tag at its commit, the worktree list, each records worktree's anchor, the records store and its layout, origin's heads and the planted record;
  - the same hub read for `FieldLocalRefs` alone with only the code store named: every other field stays empty, the records store's `Present` false and no store's `OriginRead` set;
  - a pair whose records branch removes its `.lym-anchor` marker in a commit: `FieldAnchors` has no entry for that worktree;
  - a one-repository hub (`hubforge.NewOneRepoHub`) whose records store's fetch refspec no longer maps the namespace: `FieldRecordsStore` returns an error that `errors.Is` matches to `ErrRecordsLayout` and whose message names the field and the store.
- `TestSnapshotReader_RefStorage` (table) is the one fallback test D6 names, for the worktree list and the refs.
  Each repository holds a branch, a lightweight tag and an annotated tag, and is passed as both the code and the records store, so each store's own probe is reached.
  Rows read `FieldWorktrees` and `FieldLocalRefs` alone, from the main worktree and from a subdirectory of a linked worktree:
  the `files` rows run with `PATH` emptied, so the reader starts no git process;
  the reftable rows (`git init --ref-format=reftable`, skipped below git 2.45 through `gitkit.SkipBelowGitVersion`) list the worktrees and refs through git.
  Each row wants the worktree list and the refs, the annotated tag at its commit.
  It runs serially, since it empties `PATH`, process-global state.
- `TestList_RefStorage` is deleted: its rows, the main worktree and a linked worktree's subdirectory under each backend, are the worktree-list half of `TestSnapshotReader_RefStorage`'s rows, which read through `List`.
  `TestList_ParsesPrunable`, `TestList_RejectsBareRepository` and `TestList_Errors` stay, each guarding a `List` case the reader tests do not reach.

`doc.go`'s executor section gains the reader: the phases, the fields each phase reads, the per-store ref-storage probe and the git fallback for a store whose ref storage is not `files`.

**ImpactSummary:** One new engine file reading through today's helpers with no caller-visible change; one tier1 test replaces another, two test accessors and two dead-code allowlist entries follow.

**Commit:** `15: snapshot-reader`
