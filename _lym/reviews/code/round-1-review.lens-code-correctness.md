```yaml
lens: code-correctness
outcome: reported
```

Task: review the correctness of the code the diff changes (lens: code-correctness).

**Per function or branch (case run, verdict)**

- `originNamespaceAt`
  - Missing dir: identity, not cached. Correct.
  - Dir with `.git` and a namespaced refspec: one-repo namespace, cached. Correct.
  - Broken `.git` file: error, not cached. Correct.
  - No origin remote: identity, cached. Correct.
  - Subdirectory of a repo with no `.git` entry: returns identity silently, which fails open. Every caller I checked passes a worktree root, not an anchor-joined path, so no live defect. Not a finding.
- `remoteTagRefs`: the single caller (`cleanupremotewarp.go:100`) passes a tags-relative pattern. Correct.
- `remoteBranchTips`, `lsRemoteHeads`, `remoteHeadTip`, `originBranchHead`, `resolveArchiveTip`, `remoteTagsUnder`, `fetchWeftBranchTip`: pattern is `ns.Branch` or `ns.Tag`, results map back through `LocalRef`, and `ls-remote` tail matching is handled by exact-ref comparison. Correct. `ls-remote` with no match exits 0.
- `DeleteRemoteBranch` and `DeleteRemoteBranchLeased`
  - Absent ref: `ls-remote` finds nothing, nothing is pushed, returns (false, nil). Correct.
  - Leased delete: lease and delete use the full namespaced name. Correct.
- `UpdateRemoteBranchLeased` with an empty `expectSHA` builds `--force-with-lease=ref:`, which is a lease on absence. Correct.
- `namesRef` and `retryablePushRefusal` on a namespaced destination: mapped back through `LocalRef`. Correct, including the tag case.
- `remoteTrackingRef` and `UpstreamSHA`
  - Merge ref `refs/lym/heads/b` with the namespaced refspec: maps to `refs/remotes/origin/b`. Correct.
  - Narrow or unmapped refspec: now returns no upstream, as git does. Correct.
- `openGoGitWithUnvalidatedBranchConfig` and `decodeConfigWithoutBranchValidation`
  - Plain repo, bare repo, linked worktree (relative and absolute `commondir`), and a non-repo directory (`ErrRepositoryNotExists`): correct.
  - Namespaced merge value: kept in both `Branches` and `Raw`. Correct.
- `primaryWeftBranch`
  - Key set: returns the key's value. Correct.
  - Key unset: `git config --default ""` exits 0 with empty output, so it returns the named way-forward refusal. Correct.
  - Config unparsable: `git config` fails and the cached namespace does not mask it. Correct.
  - Two-repo hub: unchanged. Correct.
- `readOriginLayout` and `refuseOneURLClone`
  - Empty origin: parses as an empty layout. Correct.
  - `lym-bolt` holds in both modes. Correct.
  - Board present with `--single-repo`: refused before the probe clone. Correct.
  - Origin whose default branch holds `.lym-warp`: refused. Correct.
  - Origin whose default branch tracks `.lym-anchor` or `_lym`: refused. Correct.
- `CloneHub` one-repo branch
  - Seed: `git init`, refspecs set before any fetch, prime key written, board orphan-created on `main`. Correct.
  - Join: fetch, `suffixWeftPrimaryBranch` adopts `origin/<prime>-weft`, board adopts `origin/main`. Correct.
  - `observedTip` comes from `layout.heads[primeRecordsBranch]`. Correct.
- `pushCloneSeed`
  - Records push fails: `fail(..., "")`, hub removed, nothing to restore. Correct.
  - Board lease lost with nothing observed: records branch deleted, leased on the pushed tip. Correct.
  - Board lease lost with a leftover observed: records branch restored to the observed tip, objects fetched beforehand. Correct.
  - Restore itself fails: both errors reported. Correct.
- `seedPushCause` and `seedRefusedByOrigin`: a transient `(failed)`, `cannot lock ref`, or `reference already exists` maps to rerun; any other `[remote rejected]` maps to the refused-host way forward. Correct.
- `refuseUnseededBoard`: two-repo hub returns nil; a joined or seeded board has an upstream so it passes. Correct.
- `AbandonSeededClone` and `PushCloneBoard`: no seed mark returns the cause unchanged; with the mark the hub is torn down. Correct.

**Findings**

1. **LOW / design** — `internal/fabricengine/destroy.go:1789-1795`, the upstream-config loop at the end of `pushCloneSeed`.
   - The board and records branch have already landed on origin when `git config branch.main.{remote,merge}` fails.
   - The function returns a bare error with no teardown, no way forward, and without going through `AbandonSeededClone`.
   - The hub is left with a board that has no upstream. A later push is then refused by `refuseUnseededBoard`, whose way forward (`--single-repo` again with `--reset`) is wrong, because origin now holds the board and `--single-repo` is refused.
   - Fix: name the right way forward in that error, which is a plain `lym fabric clone <url>` to join, or set the two `branch.main.*` keys by hand.
   - A fresh `git init` store makes a config-write failure near-impossible, so this is LOW.

No other defect found in this lens.

**Tool calls**
1. One `grep` and `sed` bash call covering `remoteTagRefs` callers, `weftPath` assignments, and `RecordsRepoRoot`. The manifest lacked all three, to check for a stale caller and a sub-directory passed to `originNamespaceAt`.
