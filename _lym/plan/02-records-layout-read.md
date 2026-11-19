# Card 2 — records-layout-read

**Create:**
- `plan:internal/fabricengine#RecordsNamespaceRoot` -> `const RecordsNamespaceRoot = "refs/lym/"`
- `plan:internal/fabricengine#RecordsNamespaceHeads` -> `const RecordsNamespaceHeads = "heads/"`
- `plan:internal/fabricengine#RecordsNamespaceTags` -> `const RecordsNamespaceTags = "tags/"`
- `plan:internal/fabricengine#OneRepositoryBoardBranch` -> `const OneRepositoryBoardBranch = "main"`
- `plan:internal/fabricengine#PrimeRecordsBranchKey` -> `const PrimeRecordsBranchKey = "lym.primeRecordsBranch"`
- `plan:internal/fabricengine#oneRepositoryNamespace` -> `func oneRepositoryNamespace() gitrepo.RefNamespace`
- `plan:internal/fabricengine#namespaceAnswers` -> `var namespaceAnswers sync.Map`
- `plan:internal/fabricengine#originNamespaceAt` -> `func originNamespaceAt(dir string) (gitrepo.RefNamespace, error)`
- `plan:internal/fabricengine#TestOriginNamespaceAt` -> `func TestOriginNamespaceAt(t *testing.T)`

**Extends:**
- `internal/fabricengine/doc.go`

**Uses:**
- `plan:internal/gitrepo#RefNamespace`
- `plan:internal/gitrepo#Repo.RemoteFetchRefspecs`

**Intent:**
Add fabric's one-repository layout constants (D1) and the one layout read (D2), in a new file of `internal/fabricengine`.

Constants, the only spelling of the layout in the tree:
`RecordsNamespaceRoot` (`refs/lym/`), `RecordsNamespaceHeads` (`heads/`), `RecordsNamespaceTags` (`tags/`),
`OneRepositoryBoardBranch` (`main`, the board's fixed branch) and `PrimeRecordsBranchKey` (`lym.primeRecordsBranch`).
`oneRepositoryNamespace()` returns `gitrepo.RefNamespace{Heads: RecordsNamespaceRoot+RecordsNamespaceHeads, Tags: RecordsNamespaceRoot+RecordsNamespaceTags}`.

`originNamespaceAt(dir)` returns the remote ref namespace of the repository whose worktree is at `dir`, read from its `origin` fetch refspecs:
- a refspec whose source side starts with `refs/lym/heads/` answers `oneRepositoryNamespace()`;
- otherwise the identity namespace, the zero value.
  A code store keeps git's default refspecs, so the read answers identity there and fabric's translation never touches it.
- `dir` that does not exist, or holds no `.git` entry, answers the identity namespace, logs nothing and is not cached,
  so a store created there later is read then.
- A repository with no `origin` remote answers identity and is cached; such a store pushes nothing.
- Any other failure of the config read is returned as an error, never an identity answer, and is not cached.
- A successful read of an existing store is cached in `namespaceAnswers`, keyed by the cleaned path, for the process;
  a later call for the same path runs no git and opens no config.
  The answer cannot change for a store, since only a clone writes the refspecs, before the store's first read.

Card 3 and card 4 are its callers; until then it is reached only by its test.

`TestOriginNamespaceAt` (tier1, internal test; it swaps the process-global log output and reads the process-wide cache, so it is not parallel) rows:
- a missing path and a directory without `.git` answer identity and log nothing;
- a store cloned with default refspecs answers identity;
- a store configured with `+refs/lym/heads/*:refs/remotes/origin/*` answers `oneRepositoryNamespace()`;
- a second read of that store after its fetch refspec is rewritten behind fabric's back still answers the namespaced value, showing the read was not repeated;
- a path read while missing, then created as a namespaced store, answers the namespaced value;
- a repository with no `origin` remote answers identity, and a second read after an `origin` with the namespaced refspec is added still answers identity, showing the answer was cached;
- a store whose `.git` is an empty file answers an error; once it is replaced by a namespaced store, the next read answers the namespaced value, showing the failed read was not cached.

`doc.go` gains the layout section: the constants, the refspec as the layout, and the cached quiet read.

**ImpactSummary:** Adds constants, one unexported read and its cache, with no production caller until cards 3 and 4; `doc.go` gains a section and no existing behavior changes.
