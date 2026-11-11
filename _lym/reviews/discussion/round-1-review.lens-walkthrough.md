```yaml
lens: walkthrough
outcome: reported
```

Task: walkthrough lens. I ran each decision's mechanism end to end on one concrete case and checked the end state against the decision's stated purpose.

**Decision walkthroughs**
- **D1, purpose not met (two cases).** Case 1, replace commits changed bytes: a tapestry run past approval with two added entries; the parent hashes the record, folds the entries, runs `replace` and gets a trailer commit. This holds. Case 2, the same bytes are already at HEAD (a sweep, or a re-run after a partial landing): the trailer commit is never made, so D2's cut has nothing to key on. A further gap: a live Merriam or other spawned role gets a notice it cannot act on.
- **D2, purpose met, given a trailer commit.** Case: v1 is the Discussion version, v2 adds A, v3 is a replace that drops A with trailer sha(v3), v4 adds B, and a writer then drops B. The history cut to `[v3]` gives no finding for A. Read as `[v4, v3]`, it gives a finding for B, as the D2 criteria expect.
- **D3, purpose met.** Case: a raddle run of promoted Sketch S, whose recipe has no Discussion approval row. `add` copies the committed record to S through `StoreDecisionRecord`, which bypasses the run lock, and D5 stamps the body hash.
- **D4, purpose not met (one case).** Case 1: Sketch S, discussed, no run; `upsert --body-file --record-file` writes both. A later promote and raddle Record-Write then read the new record. This holds. Case 2: S's earlier raddle run halted, with its worktree still on disk, and its status was unlocked by `lym batten status`. `--record-file` is admitted, but a resumed run keeps the worktree record that Record-Write never overwrites.
- **D5, purpose met.** Cases:
  - a motif approval stamps the hash, `MakeSketch` and `promoteTo` leave the body alone, so no warning;
  - a body edit, then promote, gives a warning;
  - a store whose record bytes are unchanged takes the no-write shortcut and leaves a stale stamp, so the warning stays, which is correct.
- **D6:** review-only prose; no mechanism to run.

**Findings**

1. **BLOCKING · design · D1 (with D2)**
   - Summary: "The commit is made even when the record's bytes already landed in an earlier commit, so the trailer always exists" does not hold for the seam D1 names. `CommitAnchoredPathsWithTrailers` passes straight to `CommitAnchoredPaths`, then `CommitRecordsPaths`, then `gitrepo.Repo.StageAndCommit`. That function returns `committed=false` and makes no commit when `git diff --cached --quiet` finds nothing staged. So when the bytes are already at HEAD, no trailer commit exists, D2 reads the whole history, and the dropped entries come back as `added-decision-dropped`. That breaks the second D2 acceptance criterion.
   - What exists today: an empty commit exists only as `gitrepo.Repo.CommitEmpty`, wrapped by `fabricengine`'s `commitEmptySnapshot`. That wrapper carries no caller trailers, and `CommitEmpty` refuses when the index holds staged content.
   - The record leaves to Plan how such a commit is made; that is a fact the design rests on.
   - Evidence: `internal/fabricengine/trailercommit.go:77,103`, `internal/fabricengine/commitweftpaths.go:80-81`, `internal/gitrepo/gitrepo.go:146-151`, `internal/fabricengine/recordsgit.go:178-188`, `internal/gitrepo/doc.go:466` (via grep).

2. **MEDIUM · design · D1 notice**
   - Summary: D1 says to post the notice "as `add` does", "telling the agent the record was replaced and to re-read it whole".
   - A spawned role's parent directive acts on LOOM mail only when it opens exactly `Parent decision recorded with lym loom decision add`. The record names no opening for the replace notice. With a different opening, the notice is not acted on as the parent's instruction; with the same opening, it misnames the verb.
   - `add` sends Merriam an inline notice because Merriam "may not read the record". A replace notice carries no content, so a live Webster session gets told to re-read a record it cannot read, and the folded design never reaches it.
   - Evidence: `internal/loomcli/decision.go:308-311,313-321,354-359`, `contracts/stencils/parent/parent-directive-parent.md:6,23`.

3. **MEDIUM · design · D4**
   - Summary: D4's guard keys on the entry's run status and a live batten run. The record a raddle run reads, though, is the copy in its worktree.
   - Record-Write writes the record only when it is missing ("A file that exists is never overwritten"). So after a halted raddle run's status is unlocked with `lym batten status` (the issue #116 situation), `--record-file` is admitted, and a resumed run still splits by the stale worktree record.
   - D4's stated purpose, "raddle reads the record it has always read", is missed in that case, and the record says nothing about it.
   - Evidence: `internal/loomshed/recordwrite.go:120-124,153-157`, `internal/boardengine/runlock.go:89-95` (the unlock way forward), `decision-record.md` D4 bullet 2.

4. **LOW · design · D1 commit failure**
   - Summary: "When the commit fails, it restores the prior bytes and refuses". But `StageAndCommit` can return an error after `git commit` has landed, for example when `CurrentSHA` fails.
   - Restoring the prior bytes then leaves a working copy that differs from HEAD. D1's own guard then refuses every later `replace` and `add`, and no way forward is named for that state.
   - `add` handles the same case by checking HEAD first (`afterFailedCommit`); D1 states no such check.
   - Evidence: `internal/gitrepo/gitrepo.go:157-166`, `internal/loomcli/decision.go:650-680`.

Tool calls: none.
