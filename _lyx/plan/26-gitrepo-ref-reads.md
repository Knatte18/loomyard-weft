# Card 26 — gitrepo-ref-reads

**Create:**
- `plan:internal/gitrepo#ErrRefNotFound` -> `var ErrRefNotFound = errors.New("gitrepo: ref not found")`
- `plan:internal/gitrepo#Repo.BranchExists` -> `func (r *Repo) BranchExists(branch string) (bool, error)`
- `plan:internal/gitrepo#Repo.RefSHA` -> `func (r *Repo) RefSHA(ref string) (string, error)`
- `plan:internal/gitrepo#Repo.HeadRef` -> `func (r *Repo) HeadRef() (branch string, detached bool, err error)`
- `plan:internal/gitrepo#Repo.RefTree` -> `func (r *Repo) RefTree(ref string) (string, error)`
- `plan:internal/gitrepo#Repo.Upstream` -> `func (r *Repo) Upstream(branch string) (sha string, ok bool, err error)`
- `plan:internal/gitrepo/internal/gitoracle#BranchExists` -> `func BranchExists(t testing.TB, dir, branch string) bool`
- `plan:internal/gitrepo/internal/gitoracle#RefSHA` -> `func RefSHA(t testing.TB, dir, ref string) (string, bool)`
- `plan:internal/gitrepo/internal/gitoracle#HeadRef` -> `func HeadRef(t testing.TB, dir string) (branch string, detached bool, err error)`
- `plan:internal/gitrepo/internal/gitoracle#RefTree` -> `func RefTree(t testing.TB, dir, ref string) (string, error)`
- `plan:internal/gitrepo/internal/gitoracle#Upstream` -> `func Upstream(t testing.TB, dir, branch string) (string, bool)`

**Edit:**
- `internal/gitrepo/parity_test.go`

**Uses:**
- `internal/gitrepo#readGoGit`

**Prosa:**
- `internal/gitrepo/doc.go`

**Intent:** D12's ref reads, every one through `readGoGit`.
`BranchExists` resolves `refs/heads/<branch>`; `RefSHA` resolves a full ref name to its hash, returning `ErrRefNotFound` wrapped with the ref for an absent one; `HeadRef` reads HEAD: a symbolic HEAD answers its short branch name with `detached` false whether or not the branch has a commit, so an unborn branch answers its name, and a hash HEAD answers `("", true, nil)`, which is what `rev-parse --abbrev-ref HEAD` prints as `HEAD`; `RefTree` resolves a ref or hash to a commit and returns its tree hash; `Upstream` reads `branch.<name>.remote` and `branch.<name>.merge` from the repository config and resolves `refs/remotes/<remote>/<merge short name>`, answering `ok` false when either key or the remote ref is absent.
The oracle reimplements each on `rev-parse`: `--verify --quiet refs/heads/<branch>` for existence, `<ref>` for the SHA, `--abbrev-ref HEAD` with the `branch --show-current` fallback for the unborn case, `<ref>^{tree}` for the tree and `<branch>@{upstream}` for the upstream, classifying the exit codes git documents.
`TestParity` gains rows: branch exists and missing; ref SHA for a branch, a tag and a missing ref; HEAD on a branch and detached; tree of HEAD and of a branch; upstream with and without one configured.
`TestParity_UnbornHEAD` gains the `HeadRef` row on the unborn branch and the `Upstream` row there.
The doc's read-surface list names the five reads.

**ImpactSummary:** Five new methods with no callers until cards 29 and 30; no existing method changes.
