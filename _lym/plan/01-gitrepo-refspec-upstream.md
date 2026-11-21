# Card 1 — gitrepo-refspec-upstream

**Create:**
- `internal/gitrepo#remoteTrackingRef`

**Edit:**
- `internal/gitrepo#Repo.Upstream`
- `internal/gitrepo#Repo.UpstreamSHA`
- `internal/gitrepo_test#TestUpstreamSHA`
- `internal/gitrepo_test#TestParity`

**Extends:**
- `internal/gitrepo/doc.go`

**Intent:**
Make `gitrepo`'s upstream reads follow the remote's fetch refspecs, so a records branch whose upstream names a namespaced origin ref (D2) reads its tracking ref.
`gitrepo` learns no layout: it follows whatever refspecs the store configures.

`Repo.Upstream` and `Repo.UpstreamSHA` today build the tracking ref as `refs/remotes/<remote>/<merge.Short()>`,
and `UpstreamSHA` answers `ErrNoUpstream` for a `branch.<b>.merge` outside `refs/heads/`.
A one-repository records branch has `branch.<b>.merge` = `refs/lym/heads/<b>`, which both get wrong.
Both now resolve the tracking ref with `remoteTrackingRef`, which maps the merge ref through the remote's configured fetch refspecs,
the mapping git itself uses for `@{upstream}`.
A merge ref no fetch refspec maps answers as no upstream (`ok` false, `ErrNoUpstream`), as a missing upstream does today.
A remote with no configured refspecs keeps today's `refs/remotes/<remote>/<short>` answer, so a two-repository store reads as before.

Tests:
- `TestUpstreamSHA` gains rows:
  - a store whose fetch refspec maps `refs/lym/heads/*` and whose branch merges `refs/lym/heads/<b>` answers the tracking ref's SHA;
  - the same store with a branch merging `refs/heads/<b>`, which no refspec of that store maps, while `refs/remotes/origin/<b>` exists, answers `ErrNoUpstream`;
    today's code returns that tracking ref's SHA, so the row fails on it;
  - a store whose remote has no fetch refspec configured (`git config --unset-all remote.origin.fetch`) answers `refs/remotes/origin/<b>`'s SHA, today's fallback.
- `TestParity` gains a scenario with that namespaced upstream, so `Repo.Upstream` is compared against git's own `@{upstream}`.

`doc.go` gains a paragraph on upstream reads following the fetch refspecs.

**ImpactSummary:** `Upstream`/`UpstreamSHA` change only for a merge ref outside `refs/heads/`, which no two-repository store configures, so their existing callers in fabric read the same answers.
