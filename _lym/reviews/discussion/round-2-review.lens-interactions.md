```yaml
lens: interactions
outcome: reported
```

Lens: interactions. I checked each pair of decisions that act on the same state, then each acceptance criterion and constraint against the decisions it touches.

**Pairs checked**

- D1 (board branch fixed as `main`) × D3 (`--single-repo` seeds "as a fresh two-repository bind seeds its weft"; a one-URL clone "checks out the board"): **broken**. Finding 1.
- D1 (records store configured with D1's refspecs before its first fetch) × D3 (seeding "as a fresh bind", which runs `cloneRepo`, a default clone): held. D1's general sentence binds the `--single-repo` path too, though D3 does not repeat it.
- D2 (layout read; a missing store is not cached) × D3 (the clone creates the store mid-process): held. The criterion "a store created after a missing-store read reads one repository" covers it.
- D2 (records-store remote `HEAD` answers `main`) × D4 (remote cleanup): held. `cleanupRemoteWarp` reads `remoteDefaultBranch` on the code store (`cleanupremotewarp.go:87`), which D2 does not translate.
- D2 (namespace passed to every `gitrepo` operation) × D3 (seed pushes leased under `refs/lym/heads/`): held.
- D2 × D4 (deletes and archive tags by full namespaced name): held.
- D3 (rerun replaces a leftover records branch and never adopts it) × the adopt step of the two-repository bind (`suffixWeftPrimaryBranch`, `clone.go:515-524`): held. D3 states the departure explicitly.
- D3 (rerun with a new `--subpath`) × `adoptRecordedAnchor`: held. A fresh branch carries no marker, so the clone takes the create path.
- D3 (seed teardown restores the records branch) × D3 (rerun replaces the leftover) × the rerun criterion: held. A leftover exists only when the restore itself failed.
- D3 (seed push placed at the board-push step, `fabriccli/clone.go:92-96`) × the later module-config commit and the `hubreconcile.Ensure` board push: held. Both later pushes go through D2's namespace.
- D4 (`ls-remote` before the plain delete, "in both layouts") × Scope Out ("Any behaviour change to a two-repository hub beyond D7's reverts"): **broken in wording**. Finding 2.
- D5 (prime push guard, `operator_unpushed` and `protected` removed) × D6 (`ProtectedBranchRefusal` kept) × D7 (non-retried protected refusal kept): held, and the D5/D6 criterion matches.
- D5 (removes `mergestateactive.go` and the probe cache) × D7 (defers to the peer's revert when it lands first): held. The overlap is already under Open risks.
- D7 (cherry-pick status commit) × D5 (removes the merge lifecycle): held.
- D8 (builder seeds with `--single-repo`, second machine clones with one URL) × D3's detection key: held only while the builder's code default branch is `main`. Same root cause as Finding 1.
- D9 and D10 × D3's f06acf39 refusal: held.
- Constraint "`PATTERN-run-fsm` … hold unchanged" × `## PATTERN changes` (which changes `PATTERN-run-fsm`): held. "Unchanged" reads as "the same paths as a two-repository hub".
- Constraint `PATTERN-cwd-resolution` (declared text, weft-root `.lym-anchor`) × D1 and D5: held.

**Findings**

1. **BLOCKING, design**, `_lym/discussion/decision-record.md:113`
   - Summary: D3's two clone paths both reuse the two-repository bind. That bind names the board after the code store's checked-out branch, which contradicts D1's fixed board name `main`. In a repository whose code default branch is not `main`, D1's detection key is never written or never found.
     - `--single-repo` seeds its board on that branch and pushes it to `refs/lym/heads/&lt;branch&gt;`. A later one-URL clone finds no `refs/lym/heads/main` and takes the two-repository path.
     - A one-URL clone of an existing hub looks for `origin/&lt;branch&gt;`, finds nothing, and orphan-creates a fresh board instead of checking out `main`.
   - Evidence: `internal/fabricengine/clone.go:328` (`ensureBoardWorktree(weftPath, warpBranch, boardDir)`); `internal/fabricengine/boardweft.go:24-51` (adopts `origin/&lt;warpBranch&gt;`, otherwise creates an orphan); grep finds no board-name constant in fabric apart from f06acf39's `BoltBranchName` (`onerepository.go:21`), which D5 removes. Decision record lines 42-43 (D1) and lines 109-110 and 113 (D3).
   - Fix: In D3, state that both clone paths create or check out the board on D1's board branch constant `main`, never on the code store's checked-out branch. State that this is where the one-repository clone departs from the two-repository bind. Add an acceptance criterion: a `--single-repo` clone of an origin whose default branch is not `main` puts the board at `refs/lym/heads/main`, and a one-URL clone of it checks that board out.

2. **LOW, consistency**, `_lym/discussion/decision-record.md:181`
   - Summary: D4 adds an `ls-remote` before the plain records-branch delete "in both layouts" and moves the two-repository delete to the full `refs/heads/` name. Scope Out forbids any two-repository behaviour change beyond D7's reverts, and D7 does not list this one. This is a new non-BLOCKING finding, so the incremental rule may drop it.
   - Evidence: decision record lines 28 and 181-183; `internal/gitrepo/push.go:313-322` (today's short-name delete detects an absent ref from git's stderr).
   - Fix: Name D4's delete change in Scope Out's exception beside D7's reverts.

Outside my lens: D3 line 143 says the refusal check is one `git ls-remote`. Two of its refusal arms read file contents of origin's default branch (`.lym-warp`, root `.lym-anchor` or `_lym`), which `ls-remote` cannot see. Today that read needs a probe clone (`recordsprobe.go:61-74`).

**Tool calls**

1. `grep` for `"main"`, `BoardBranchName` and `BoltBranchName =` in `internal/fabricengine` and `internal/fabriccli`, to confirm whether fabric names the board with a constant. My context held only the call to `ensureBoardWorktree`.
