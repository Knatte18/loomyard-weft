```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: I ran each decision's mechanism end to end against the code it would meet.

**Findings**

1. **BLOCKING — D2 and D4: the translation misses flag-filtered listings and does not map output back, so records-store reads return code branches.**
   - D2 translates only "ref names fabric spells" going *to* origin. Several records-store reads select refs by flag instead and parse the reply by the `refs/heads/` prefix.
   - `originreconcile.go` `lsRemoteHeads` runs `ls-remote --heads origin` for the records repository and keeps only lines starting with `refs/heads/`.
   - `cleanupremotewarp.go` does the same: `remoteBranchTips` runs `ls-remote --heads origin` and strips `refs/heads/`, and `remoteTagRefs` runs `ls-remote --tags`.
   - `remoteleftover.go` and `archive.go` run `ls-remote --heads origin refs/heads/<branch>`.
   - Walkthrough on a one-repository hub: origin reconcile on the records store gets every *code* branch back as a records head, and no `refs/lym/heads/*` line. `--heads` filters output to `refs/heads/`, so even a translated pattern `refs/lym/heads/x` returns nothing. The weft-branch existence checks in `archive.go` and `remoteleftover.go` then see "absent".
   - `remoteDefaultBranch` (`ls-remote --symref origin HEAD`) returns the code repository's default branch.
   - D4's purpose, one code path per operation running correctly through the translation, is not met. The record needs to say that the translation covers listing flags (`--heads` and `--tags`) and the reverse mapping of listed names, not only names fabric sends. It must also say what a records-store `HEAD` symref read means.

2. **MEDIUM — D3: the refused-push teardown does not fit the reused clone flow.**
   - D3 reuses the two-repository clone, then says a namespace push that origin refuses removes the hub directory.
   - In the reused flow, the first push to origin is `CloneAndWire`'s `b.PushRecorded` for the board. It runs after the hub-wide config commit. On error, `CloneAndWire` leaves the clone intact and points to `lym fabric reconcile`, by its own doc comment.
   - Walkthrough: `--single-repo` on a host that rejects `refs/lym/` fails at `PushRecorded`, and the hub directory stays. That contradicts D3 and its acceptance criterion.
   - The record needs to place a dedicated seed push, with teardown, before `CloneAndWire`'s board push. Alternatively, it can state that a one-repository clone overrides the left-intact rule at that push.

3. **MEDIUM — D7: the deferred-mail revert describes the old behaviour wrongly, and its criterion cannot detect the revert.**
   - The exploration found that before f06acf39, `FailedPushesOwedBy` and `recordPushFailure` skipped *every* `Deferred` record. f06acf39 narrowed the skip to deferred records that name no commits.
   - D7 says "a deferred record with no commits is not mailed, as before f06acf39". That statement holds under f06acf39 too.
   - The D7 acceptance criterion tests only the no-commit case, which passes with or without the revert.
   - To revert, the decision and criterion need to cover a deferred record that does name commits, which then also sends no mail.

4. **LOW — D1 and D3: creating the records store as a plain clone defeats the small-store aim.**
   - D3 says the clone "makes the records store as a clone of the same URL with D1's refspecs".
   - A plain `git clone` cannot take a custom fetch refspec, and today's `cloneRepo` is a plain clone. Walkthrough: it fetches every code branch into the records store before the reconfigure, and leaves `refs/remotes/origin/<code branch>` refs that D2's listings could later read.
   - The record should say the store is initialised with the refspecs before its first fetch, or pruned afterwards.

5. **LOW — D3: `--single-repo` does not refuse an existing two-repository hub's code origin.**
   - Walkthrough: the code origin has no namespace, and its default branch holds no `.lym-warp`, so the clone proceeds and starts a second hub on that repository.
   - f06acf39 documented this as a hazard. D3 names no refusal and D9 names no hazard for it.

**Per decision**

| Decision | Case walked | Purpose met |
|---|---|---|
| D1 | Default code-store clone and fetch skip `refs/lym` | Met |
| D2 | Origin reconcile on the records store | **Not met** (finding 1) |
| D3 | `--single-repo` against a host that rejects custom refs | **Not met** (finding 2) |
| D3 | One-URL clone of a namespaced origin | Met, given finding 4 |
| D4 | Teardown deletes `refs/lym/heads/<branch>-weft` and pushes an archive tag | Delete and tag push met; the existence checks before them are broken (finding 1) |
| D5 | Merge in a one-repository hub: plain two-repository merge, no strip | Met |
| D6 | Landing push refused as protected: named, not retried | Met |
| D7 | Status commit during a code-worktree cherry-pick commits | Met |
| D7 | Deferred mail | **Not met** (finding 3) |
| D8 | Builder over one bare origin | Met, provided D3's flow works |
| D9 | Docs | Met; one hazard missing (finding 5) |
| D10 | Old-layout hub re-cloned | Met |

**Tool calls**
- Grepped `internal/fabricengine` and `internal/gitrepo` for `ls-remote`, `--heads`, `--tags`, `--prune` and `--all`. My context held only explicit-refspec fetch and push sites, not flag-filtered listings.
- Read `remoteBranchTips`, `remoteTagRefs` and `remoteDefaultBranch` in `cleanupremotewarp.go`, and `lsRemoteHeads` in `originreconcile.go`, to confirm how they parse output. My context named those files only by summary.
