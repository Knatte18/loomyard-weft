```yaml
lens: walkthrough
outcome: reported
```

The walkthrough lens was run against each decision, executing every step on the data the step before it produces.

**Walkthrough list**
- D1, records sibling: a one-URL clone on a second machine where the code default branch is `master`. Purpose **not met**: the board is checked out as a fresh orphan `master` instead of `origin/main` (F-W1).
- D2, a records-branch push and upstream write in a one-repository hub: purpose met as stated. The namespace is passed to `gitrepo` push, lease, result matching and the upstream write.
- D3 `--single-repo`, over a fresh origin whose code default branch is `master`. Purpose **not met**:
  - the fresh-bind seed puts the board on `master` locally, while D1 and the push name `refs/lym/heads/main` (F-W1);
  - a plain weft clone of the code URL would fetch code refs and start the records branch at the code tip (F-W1).
- D3 refusals, a one-URL `--single-repo` clone of an f06acf39 origin with the `lym-bolt` branch deleted but `.lym-anchor` still tracked. Purpose **not met**: the stated single `ls-remote` cannot see file contents, so the content arms have no input (F-W2).
- D3 seed push, `--single-repo` with the code default branch `main`. Purpose met:
  - the push replaces `b.PushRecorded` at `CloneAndWire`'s board-push step;
  - the records branch holds the anchor commit, and the board goes last;
  - `CloneResult` carries the hub token the teardown needs.
- D3 rerun after a partial seed, `--single-repo` with no board on origin. Purpose met:
  - the init and refspec fetch brings a leftover `origin/&lt;prime&gt;-weft`, the branch is created fresh, and the anchor takes the create path;
  - the push is leased on the observed tip, so it replaces the leftover.
- D4 teardown of a never-pushed `feat-weft`: purpose met. `ls-remote` of the full name finds no ref, nothing is pushed and no deletion is recorded.
- D5 to D7, a records commit and a status commit during a cherry-pick in the records worktree: purpose met, since nothing layout-specific is left to run.
- D8, the builder seeds with `--single-repo` and the second hub uses one URL: purpose met only when the fixture's code default branch is `main`. Otherwise F-W1 applies.
- D9 and D10: nothing executable to walk beyond the D3 refusal; held.

**Findings**

F-W1 — BLOCKING, design, `_lym/discussion/decision-record.md:42` (D1's fixed board name `main`) against D3 at lines 109–114.
- Summary: D3's two clone paths hand the board checkout the code default branch rather than D1's fixed `main`. So on any code repository whose default branch is not `main`:
  - the `--single-repo` seed commits the board on a local branch other than the `refs/lym/heads/main` it must push;
  - a second machine's one-URL clone checks out an empty orphan board and hides the real one.
  - The `--single-repo` path "as a fresh two-repository bind seeds its weft" also runs `cloneRepo` on the URL with default refspecs. That fetches every code branch into the records store and starts the records branch at the code tip, against D1's rule to configure the refspecs before the first fetch.
- Evidence:
  - `internal/fabricengine/clone.go:316-331` reads `warpBranch` from the warp clone and calls `ensureBoardWorktree(weftPath, warpBranch, boardDir)`.
  - `internal/fabricengine/boardweft.go:24-51` adopts `origin/&lt;warpBranch&gt;` only when it exists and otherwise creates a fresh orphan.
  - `internal/fabricengine/clone.go:304` clones the weft with `cloneRepo` (plain `git clone`).
  - `internal/fabricengine/clone.go:521-530` creates the fresh suffixed branch at the clone's current HEAD.
- Fix: in D3, state that both clone paths create the records store by `git init` with D1's refspecs and then a fetch, never `cloneRepo`. State also that the board worktree is checked out on D1's fixed board branch, not on the warp branch. Add `master` as a code default branch to a D1 or D3 acceptance criterion.

F-W2 — BLOCKING, design, `_lym/discussion/decision-record.md:142-153` (D3's precondition check).
- Summary: D3 says the check "runs `git ls-remote` on origin once", but three of its refusal arms need the default branch's tree:
  - holds `.lym-warp`;
  - tracks a root `.lym-anchor`;
  - tracks `_lym`.
  `ls-remote` returns only ref names and tips, so as written those arms receive no input and never fire. A half-cleaned f06acf39 origin with `lym-bolt` deleted then passes `--single-repo`.
- Evidence: `internal/fabricengine/recordsprobe.go:60-120`. Today's probe reads tree content through a staged shallow `git clone --depth 1 --filter=tree:0 --no-checkout` into a temporary directory plus `git show HEAD:&lt;file&gt;`, not through `ls-remote`.
- Fix: in D3, state that the refs are read by one `ls-remote` and the tree arms by the existing staged probe clone of the default branch. Say that the probe clone runs before the hub directory is created and is removed afterwards, so "before creating anything" still holds for the hub.

Tool calls:
1. Read `_lym/reviews/discussion/round-2-changes.md` to see which hunks changed, for the incremental-round rule; my context did not hold it. It lists only the two files, with no hunks.
