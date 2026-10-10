# Card 25 — gitrepo-geometry

**Create:**
- `internal/gitrepo#Repo.geometry`
- `internal/gitrepo#Repo.GitDir`
- `internal/gitrepo#Repo.CommonDir`
- `internal/gitrepo#Repo.Toplevel`
- `internal/gitrepo/internal/gitoracle#GitDir`
- `internal/gitrepo/internal/gitoracle#CommonDir`
- `internal/gitrepo/internal/gitoracle#Toplevel`

**Edit:**
- `internal/gitrepo#Repo`
- `internal/gitrepo#Repo.MergeHeads`
- `internal/gitrepo/parity_test.go`
- `cmd/lyx/gitrepoboundary_test.go`

**Uses:**
- `internal/dotgit#Read`
- `internal/dotgit#FindRoot`

**Prosa:**
- `internal/gitrepo/doc.go`
- `pattern/PATTERN-gitrepo-client-boundary.md`
- `PATTERN.md`

**Intent:** D12: `gitrepo` reads a repository's geometry through `dotgit` once per handle.
`Repo` gains a `sync.Once`, a cached `dotgit.Geometry` and its error; `geometry` fills them on first use from `dotgit.Read(r.path)`, so a handle's git dir and common dir are read once for its life, and nothing else is cached, since HEAD and refs change under a process.
`GitDir` and `CommonDir` return the cached absolute directories, and `Toplevel` returns `dotgit.FindRoot(r.path)`'s root.
`MergeHeads` joins `MERGE_HEAD` onto `GitDir` instead of spawning `rev-parse --git-path`, which keeps its linked-worktree correctness, since the per-worktree git dir is where that file lives; it leaves the pinned run-bound set, so `gitrepoPinnedRunBoundMethods` in the boundary scan drops it.
The oracle gains the three reads on `rev-parse --git-dir`, `--git-common-dir` and `--show-toplevel` through `gitexec.Run`, absolutized against `dir` when git answers a relative path.
`TestParity` gains a step that adds a linked worktree and asserts the three reads agree with the oracle on the primary and on the linked worktree.
The doc's read-surface paragraph names the geometry reads and `dotgit` as the one local read beside go-git; `PATTERN-gitrepo-client-boundary.md` and the `PATTERN.md` entry say go-git and the `dotgit` geometry read own local reads.
Per the record's amendment "PATTERN overview stays under its cap", the card keeps `PATTERN.md` at most 16200 bytes (`wc -c`) and drops no invariant: the entry's added words are offset in the same line, which reads "go-git and the `dotgit` geometry read own local reads, `gitexec` remote-authenticating or tree-mutating work", and by dropping `PATTERN-comment-line-breaks`'s ", with no column limit", which `pattern/PATTERN-comment-line-breaks.md` already states.
If the file is still above 16200 bytes, the card shortens wording in another overview line whose background file already carries the detail, never one an agent needs inline.

**ImpactSummary:** `Repo` gains three fields and four methods; `MergeHeads` keeps its signature and result, and the one scan that counts its spawn is edited in the same card.
