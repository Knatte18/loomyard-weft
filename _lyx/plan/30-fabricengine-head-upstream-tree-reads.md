# Card 30 — fabricengine-head-upstream-tree-reads

**Edit:**
- `internal/fabricengine#Topology.Add`
- `internal/fabricengine#Topology.Checkout`
- `internal/fabricengine#Topology.switchOrForkWeft`
- `internal/fabricengine#createDormantWeftForRawWarp`
- `internal/fabricengine#readBranch`
- `internal/fabricengine#Fabric.weftHasUpstream`
- `internal/fabricengine#Fabric.warpUpstreamSHA`
- `internal/fabricengine#upstreamSHAAt`
- `internal/fabricengine#checkUnlandedRemoteTip`
- `internal/fabricengine#checkUnlandedWork`

**Uses:**
- `internal/gitrepo#Repo.HeadRef`
- `internal/gitrepo#Repo.Upstream`
- `internal/gitrepo#Repo.RefTree`

**Prosa:**
- `internal/fabricengine/doc.go`

**Intent:** D12: the current-branch, upstream and tree spawns move onto `gitrepo`'s reads, and the survivors are named.
`readBranch` becomes `HeadRef`: a detached HEAD answers `HEAD`, as before, and the unborn fallback goes, since `HeadRef` answers the unborn branch's name directly; `Topology.Add`, `Topology.Checkout`, `switchOrForkWeft` and `createDormantWeftForRawWarp` read the current branch the same way.
`weftHasUpstream`, `warpUpstreamSHA` and `upstreamSHAAt` become `Upstream` of the current branch read through `HeadRef`, with `ok` false where a non-zero exit meant "no upstream or detached".
`checkUnlandedRemoteTip` and `checkUnlandedWork` read the parent's tree through `RefTree`; the `merge-tree --write-tree` spawn beside it stays, since it computes a merge.
Surviving `rev-parse` spawns, listed in the package doc with their reason: `InstallPostCheckoutHook`'s `--git-path hooks`, which honours `core.hooksPath`; `resolveArchiveTip`'s `FETCH_HEAD^{commit}`, which resolves git's fetch output through revision syntax; and websterengine's `--disambiguate` and `<commit>:<path>` probes, which are git's own disambiguation.
Pure refactor behind unchanged signatures; the fabricengine integration tier, `hook_test.go`'s `core.hooksPath` case included, covers every site.

**ImpactSummary:** Ten fabricengine functions change body; no signature changes and no caller outside the package is affected.

**Verify:** lyx gate test --tags integration ./internal/fabricengine ./internal/fabriccli
