# Card 1 — gitrepo-ref-namespace

**Create:**
- `plan:internal/gitrepo#RefNamespace` -> `type RefNamespace struct`
- `plan:internal/gitrepo#RefNamespace.Branch` -> `func (n RefNamespace) Branch(name string) string`
- `plan:internal/gitrepo#RefNamespace.Tag` -> `func (n RefNamespace) Tag(name string) string`
- `plan:internal/gitrepo#RefNamespace.LocalRef` -> `func (n RefNamespace) LocalRef(ref string) (string, bool)`
- `plan:internal/gitrepo#Repo.RemoteFetchRefspecs` -> `func (r *Repo) RemoteFetchRefspecs(name string) ([]string, error)`
- `plan:internal/gitrepo#remoteTrackingRef` -> `func remoteTrackingRef(cfg *config.Config, remote string, merge plumbing.ReferenceName) (plumbing.ReferenceName, bool)`
- `plan:internal/gitrepo_test#TestRefNamespace` -> `func TestRefNamespace(t *testing.T)`
- `plan:internal/gitrepo_test#TestRemoteFetchRefspecs` -> `func TestRemoteFetchRefspecs(t *testing.T)`

**Edit:**
- `internal/gitrepo#Repo.Upstream`
- `internal/gitrepo#Repo.UpstreamSHA`
- `internal/gitrepo_test#TestUpstreamSHA`
- `internal/gitrepo_test#TestParity`

**Extends:**
- `internal/gitrepo/doc.go`

**Intent:**
Give `gitrepo` the remote ref namespace type that D2 passes to every records-store remote operation, and make its upstream reads follow the remote's fetch refspecs.
`gitrepo` learns no layout: it receives a namespace and maps names through it.

`RefNamespace` has two string fields, `Heads` and `Tags`, the origin-side prefixes of branches and tags.
Its zero value is the identity namespace of a two-repository hub, `refs/heads/` and `refs/tags/`.
Fabric builds the one-repository value from its own constants (card 2); `gitrepo` holds no `refs/lym/` literal.
- `Branch(name)` returns the origin ref of a branch: `Heads+name`, or `refs/heads/<name>` for an empty `Heads`.
- `Tag(name)` does the same for a tag.
- `LocalRef(ref)` maps an origin ref name read back from `ls-remote` or a push result to its local `refs/heads/` or `refs/tags/` name.
  A ref outside both prefixes answers false, so a one-repository listing never reads a code ref as a records ref.

`Repo.RemoteFetchRefspecs(name)` returns the configured `remote.<name>.fetch` refspecs, read through go-git's local config like `RemoteURL`, spawning no git.
A remote that is not configured returns an error wrapping `ErrRemoteNotConfigured`; any other config read failure is returned as it is.
Fabric's layout read (card 2) is its one caller.

`Repo.Upstream` and `Repo.UpstreamSHA` today build the tracking ref as `refs/remotes/<remote>/<merge.Short()>`,
and `UpstreamSHA` answers `ErrNoUpstream` for a `branch.<b>.merge` outside `refs/heads/`.
A one-repository records branch has `branch.<b>.merge` = `refs/lym/heads/<b>`, which both get wrong.
Both now resolve the tracking ref with `remoteTrackingRef`, which maps the merge ref through the remote's configured fetch refspecs,
the mapping git itself uses for `@{upstream}`.
A merge ref no fetch refspec maps answers as no upstream (`ok` false, `ErrNoUpstream`), as a missing upstream does today.
A remote with no configured refspecs keeps today's `refs/remotes/<remote>/<short>` answer, so a two-repository store reads as before.

Tests:
- `TestRefNamespace` (tier0) is a table over the identity and a namespaced value: `Branch`, `Tag`, and `LocalRef` of a namespaced head, a namespaced tag, and a ref outside both, which answers false.
- `TestRemoteFetchRefspecs` (tier1) rows: a default clone answers `+refs/heads/*:refs/remotes/origin/*`;
  a store configured with `+refs/lym/heads/*:refs/remotes/origin/*` and `+refs/lym/tags/*:refs/tags/*` answers both;
  an unconfigured remote name returns an error matching `ErrRemoteNotConfigured`.
- `TestUpstreamSHA` gains rows:
  - a store whose fetch refspec maps `refs/lym/heads/*` and whose branch merges `refs/lym/heads/<b>` answers the tracking ref's SHA;
  - the same store with a branch merging `refs/heads/<b>`, which no refspec of that store maps, while `refs/remotes/origin/<b>` exists, answers `ErrNoUpstream`;
    today's code returns that tracking ref's SHA, so the row fails on it;
  - a store whose remote has no fetch refspec configured (`git config --unset-all remote.origin.fetch`) answers `refs/remotes/origin/<b>`'s SHA, today's fallback.
- `TestParity` gains a scenario with that namespaced upstream, so `Repo.Upstream` is compared against git's own `@{upstream}`.

`doc.go` gains a paragraph on `RefNamespace` and on upstream reads following the fetch refspecs.

**ImpactSummary:** Adds a type and a config read with no production caller yet; `Upstream`/`UpstreamSHA` change only for a merge ref outside `refs/heads/`, which no two-repository store configures, so their existing callers in fabric read the same answers.
