# Card 29 — fabricengine-branch-and-ref-reads

**Edit:**
- `internal/fabricengine#ensureBoardWorktree`
- `internal/fabricengine#Topology.Add`
- `internal/fabricengine#suffixWeftPrimaryBranch`
- `internal/fabricengine#bornWeftPrimaryBranch`
- `internal/fabricengine#refuseUncheckedOutWarpClone`
- `internal/fabricengine#probeWeftBinding`
- `internal/fabricengine#deleteWarpBranch`
- `internal/fabricengine#localBranchExists`
- `internal/fabricengine#weftBranchExists`
- `internal/fabricengine#revParseBranch`
- `internal/fabricengine#recordTrailerSHA`
- `internal/fabricengine#probeLiveLocalWeft`
- `cmd/lyx/checkedcall_test.go`

**Uses:**
- `internal/gitrepo#Repo.BranchExists`
- `internal/gitrepo#Repo.RefSHA`
- `internal/gitrepo#ErrRefNotFound`
- `internal/gitrepo#Repo.CurrentSHA`

**Intent:** D12: fabricengine's branch-existence, HEAD-existence and ref-SHA spawns move onto `gitrepo`'s reads.
Branch existence (`rev-parse --verify [--quiet] refs/heads/<b>`) becomes `BranchExists` in `ensureBoardWorktree`, `Topology.Add`'s leftover-branch check, `bornWeftPrimaryBranch`, `deleteWarpBranch`, `localBranchExists` and `weftBranchExists`; `suffixWeftPrimaryBranch`'s remote-ref probe becomes `RefSHA` on `refs/remotes/origin/<suffixed>` with `ErrRefNotFound` as "absent".
HEAD existence (`rev-parse --verify --quiet HEAD`) in `refuseUncheckedOutWarpClone` and `probeWeftBinding` becomes `CurrentSHA` with `ErrNoCommits` as "unborn".
A ref's SHA (`rev-parse refs/heads/<b>`, with or without `^{commit}`) in `Topology.Add`'s pushed-tip read, `revParseBranch`, `recordTrailerSHA` and `probeLiveLocalWeft` becomes `RefSHA`.
Each site keeps its error classification: where a non-zero exit was an answer, `ErrRefNotFound` or `ErrNoCommits` is that answer, and any other error is wrapped as before.
`weftBranchExists` loses its raw-site marker, so `checkedCallPinnedRawSites` for `internal/fabricengine` drops to zero.
Pure refactor behind unchanged signatures; the fabricengine integration tier covers every site.

**ImpactSummary:** Twelve fabricengine functions change body; no signature changes and no caller outside the package is affected.

**Verify:** lyx gate test --tags integration ./internal/fabricengine ./internal/fabriccli
