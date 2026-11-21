```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens. Map each change to the decision that asks for it, and each decision to the changes that build it.

**Per change → decision**

| Change | Traces to |
|---|---|
| `gitrepo/refnamespace.go`, and the namespace parameter on push, lease, retry, delete, `namesRef` and `pushdestination.go` | D2, D4 (plan cards 2 and 3) |
| `DeleteRemoteBranch` lists the full ref first and drops `remoteRefAbsentTrigger` | D4 |
| `UpdateRemoteBranchLeased` accepts an empty lease, meaning absence | D3 (the board push is leased on absence) |
| `RemoteFetchRefspecs`; `Upstream` and `UpstreamSHA` follow the fetch refspecs (`remoteTrackingRef`) | D2 and Added "Tolerant config read" (card 1) |
| `gogit.go` storer with tolerant config decode, own open logic, `go.mod` billy promoted to direct, `worktree.go` type assertion | Added "Tolerant config read" (Option C). The decision names one helper; this also replaces `PlainOpenWithOptions`, but go-git's open reads the config, so it is forced. |
| `fabricengine/recordslayout.go`: constants, `originNamespaceAt` and its cache | D1, D2; Added "lym's own records-namespace branch names are named constants" |
| `clone.go`: one-URL join, `--single-repo`, `initRecordsStore`, `readJoinedBoardRecords`, the `lym.primeRecordsBranch` key | D1, D3 |
| `readOriginLayout`, `refuseOneURLClone`, `probeCodeOriginRoot` | D3, D10 (the earlier-layout refusal) |
| `destroy.go` seed push: `pushCloneSeed`, `restoreSeededRecordsBranch`, `checkSeedPush`, `seedPushCause`, `seedPushHook` | D3 |
| `PushCloneBoard`, `AbandonSeededClone`, `CloneResult.seed` | D3 (the seed marker; `fabriccli` reads no layout) |
| `pushclassified.go` `refuseUnseededBoard` | Forced by D3. Its bound says the board lands only while origin holds none, so an unseeded board must not be pushed unleased. Card 5 names its test. |
| `fabriccli` `--single-repo` flag, help text, usage | D3 |
| `ensureBoardWorktree` takes the board branch | D1, D3 |
| `primaryWeftBranch` reads the key; `cleanup.go` docs | D1 |
| Namespaced fetch and ls-remote in `add`, `archive`, `checkout`, `cleanupremotewarp`, `originreconcile`, `ownerarchive`, `remoteleftover`, `weftbranch`, `recordsgit`, `takeoverwrite`, `resetsubpathcheck`, `recordsprobe` | D2, D4 |
| Zero-namespace arguments in `handover.go` and `updateRemoteBranch` | Forced by the new gitrepo signatures. Both are code-store pushes. |
| `hubforge`: `CopyCodeBare`, `NewOneRepoHub`, the `cloneHub` flag | D8 |
| Tests: `clone_*`, `onerepository_lifecycle`, `recordslayout`, cleanup tables, gitrepo push, delete, `parity` and `refnamespace` tests; `export_test` hook | D8, D4, D2, and D3's acceptance criteria |
| Existing test call-site updates in `gitrepo` and `stencilcommit` | Forced by the signatures |
| `README.md`, `docs/getting-started.md`, package `doc.go` files | D9 (the package docs are forced) |
| `PATTERN.md`, `pattern/PATTERN-code-conventions.md`, `pattern/PATTERN-single-repo-hub.md` | D9, Added "PATTERN change", "Declare", "Add anew"; both declared entries landed with the declared text |

**Per decision → builders**
- D1: `recordslayout.go`, `clone.go` (the key and the board branch), `primaryWeftBranch`, `initRecordsStore`.
- D2: `RefNamespace`, `originNamespaceAt` and the namespaced reads and pushes across `fabricengine`.
- D3: `clone.go`, `recordslayout.go` refusals, the seed push in `destroy.go`, the `fabriccli` flag.
- D4: the namespaced push, fetch and listing sites, the `DeleteRemoteBranch` pre-listing, the cleanup and archive reads.
- D8: `hubforge` and the layout tests.
- D9: the docs and PATTERN files. The D9 text drops were already landed by 13cebe3d, as the plan's `d9-drops-already-landed` decision says.
- D10: only the earlier-layout refusal, which is all D10 asks for.
- Added "Peer merge-probe has landed": no change needed.
- Added "Tolerant config read (Option C)": `gogit.go`, `RemoteFetchRefspecs`, `Upstream` and `UpstreamSHA`. The superseded "No branch merge config" entry rightly has no builder.
- Added "PATTERN change", "Declare", "Add anew": built in `PATTERN.md` and the two background files.

**Findings**
None. No unrequested scope, and no decision left without a builder.

One point I could not settle: D2's "a remote HEAD read for the records store answers `main`, never origin's HEAD". The diff does not touch the `ls-remote --symref origin HEAD` read at `cleanupremotewarp.go:327`, so I did not confirm it never runs against a records store. The `code-correctness` lens owns that check.

Tool calls: none. The manifest already held everything this lens needed.
