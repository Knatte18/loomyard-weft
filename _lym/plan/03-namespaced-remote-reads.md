# Card 3 — namespaced-remote-reads

**Edit:**
- `internal/fabricengine#remoteHeadTip`
- `internal/fabricengine#remoteTagsUnder`
- `internal/fabricengine#remoteBranchTips`
- `internal/fabricengine#remoteTagRefs`
- `internal/fabricengine#Topology.cleanupRemoteWarp`
- `internal/fabricengine#originBranchHead`
- `internal/fabricengine#resolveArchiveTip`
- `internal/fabricengine#probeWeftLeftover`
- `internal/fabricengine#probeLiveLocalWeft`
- `internal/fabricengine#lsRemoteHeads`
- `internal/fabricengine#fetchMovedHeads`
- `internal/fabricengine#resolveWeftBranch`
- `internal/fabricengine#targetWeftTip`
- `internal/fabricengine#Topology.Add`
- `internal/fabricengine#readSideOwnerArchives`

**Extends:**
- `internal/fabricengine/doc.go`

**Uses:**
- `internal/fabricengine#originNamespaceAt`
- `internal/gitrepo#RefNamespace.Branch`
- `internal/gitrepo#RefNamespace.Tag`
- `internal/gitrepo#RefNamespace.LocalRef`

**Intent:**
Carry D2's namespace through every ref name fabric itself sends to, or reads back from, a records store's origin: `ls-remote` patterns and output, fetch sources, and tag names.
Each helper reads `originNamespaceAt(dir)` for the repository at `dir` and returns its error; in a code store, and in any two-repository store, the read answers identity and each command is the one it runs today.
No signature changes, so no caller changes.

Listings select by namespaced pattern, never by `--heads` or `--tags` (D2):
- `remoteHeadTip`, `originBranchHead` and `resolveArchiveTip` run `ls-remote origin <ns.Branch(b)>` and keep only the line naming exactly that ref.
- `remoteBranchTips` and `lsRemoteHeads` run `ls-remote origin <ns.Branch("*")>` and key each line by `ns.LocalRef`, dropping lines it does not map;
  in a one-repository store they therefore see the namespaced records branches and no code branch (D2, D4).
- `remoteTagsUnder` runs `ls-remote origin <ns.Tag(prefix)>*` and maps names back through `ns.LocalRef`, peeled lines preferred as today.
- `remoteTagRefs` takes a tag-name pattern relative to the tags prefix and runs `ls-remote origin <ns.Tag(pattern)>`, returning local `refs/tags/` names;
  `Topology.cleanupRemoteWarp` passes `archive/*` instead of `refs/tags/archive/*`, and `hasWeftEvidence` keeps parsing local `refs/tags/archive/<slug>/` names, unchanged.

Fetch sources name the origin ref:
- `probeWeftLeftover` and `probeLiveLocalWeft` fetch `ns.Branch(weftBranch)` and each archive tag's `ns.Tag(name)`;
- `resolveWeftBranch` and `targetWeftTip` fetch `ns.Branch(b):refs/remotes/origin/<b>`;
- `fetchMovedHeads` fetches `+<ns.Branch(b)>:<trackingRefOf(b)>`; its dry-run form fetches by SHA and is unchanged;
- `Topology.Add`'s records fetch reads `ns.Branch(weftBranch):refs/heads/<weftBranch>`; its code fetch is unchanged;
- `readSideOwnerArchives` fetches `ns.Tag(tag)`.

`RecordsBranchFile` fetches the SHA `remoteHeadTip` returns, so it needs no change.

No records-store path reads origin's `HEAD`: `remoteDefaultBranch` and `warpDefaultBranch` run only in the code store, so D2's remote-`HEAD` rule needs no branch here.
`doc.go` gains a paragraph on namespaced listings and fetches.
A one-repository hub exists only from card 5; card 7's tests reach these paths in it, and the existing two-repository tests pin the identity behaviour.

**ImpactSummary:** Fifteen fabricengine members change the ref strings they send or parse; every two-repository and code-store call sends the same refs as today apart from `--heads`/`--tags` giving way to an equivalent pattern, and no signature or caller changes.
