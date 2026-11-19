```yaml
lens: bound-breaker
outcome: reported
```

Task: bound-breaker lens. I attacked every bound, guarantee, ordering promise and claim about existing code in the decision record.

**Claims attacked**

- **D1, held:** a default clone fetches no records ref, and the code store's refspecs never carry a records ref.
- **D1, broken:** "local geometry is a two-repository hub's, unchanged" together with a board branch fixed as `main` while the prime's records branch is `&lt;prime-branch&gt;-weft`. This breaks the existing invariant that the board branch is the code store's primary branch (finding 1).
- **D1, broken:** "a renamed code default branch cannot hide the board". The clone path that D3 reuses still creates the board on the code store's branch, `clone.go:328`. Finding 1 covers it.
- **D2, held:**
  - a missing store reads as two repositories and is not cached;
  - an unreadable store is an error;
  - the translation acts only on the records store's origin.
- **D2, held:** "No module outside fabric and `gitrepo` reads the namespace". No counterexample inside fabric's own callers.
- **D3, held:** the detection key is pushed last, so a failed seed push leaves no board branch on origin.
- **D3, held:** the board push is leased on absence, so only one of two concurrent `--single-repo` clones lands a board.
- **D3, broken:** the records-branch replace bound "replaces … only through a push leased on the tip it observed". A clone still in flight is indistinguishable from a failed attempt's leftover (finding 2).
- **D3, broken:** "a clone whose board push fails leaves the records branch as it found it, unless another push moved it since". An unreachable origin on the restore push breaks it (finding 3).
- **D3, broken:** "The check runs `git ls-remote` on origin once … It refuses … an origin whose default branch holds `.lym-warp` … tracks a root `.lym-anchor` or `_lym`". `ls-remote` cannot read file contents (finding 4).
- **D3, held:** a two-repository weft with the legacy root `.lym-anchor` takes no `--single-repo` and holds no `lym-bolt`, so it clones as today.
- **D3, held:** `--single-repo` writes only under `refs/lym/`.
- **D4, held:** the plain delete runs `ls-remote` on the full name first, so a never-pushed branch records no deletion.
- **D4, broken (minor):** "an absent ref answers 'not deleted'". There is a gap between the `ls-remote` and the push (finding 5).
- **D4 against Scope Out, broken (minor):** "in both layouts" changes a two-repository hub's delete path, but Scope Out promises no change beyond D7 (finding 6).
- **D4, held:** an absent branch fails the leased delete's lease once D2 namespaces the lease ref.
- **D7, D10 and the Constraints, held:** no counterexample found.

**Findings**

1. **BLOCKING, design, `_lym/discussion/decision-record.md:42`.** D1's fixed board branch `main` breaks fabric's invariant that the board's branch names the prime's code branch. On a code repository whose default branch is not `main`, the orphan sweep would delete the prime's records line.
   - Evidence: `primaryWeftBranch` in `internal/fabricengine/cleanup.go:337-351` derives the prime's records branch as `&lt;board branch&gt;-weft` from the branch the Bolt checkout is on. Its comment says the board sits "on the warp's own unsuffixed default branch" and that "a wrong answer loses data" (`cleanup.go:327-330`).
   - Counterexample: a code repository whose default branch is `master`. Under D1 the board is on `main` and the prime's records branch is `master-weft`. `primaryWeftBranch` answers `main-weft`, so the sweep enumerates `master-weft` as an orphan.
   - D3 also says the clone "wires the hub as a two-repository clone does", and that path calls `ensureBoardWorktree(weftPath, warpBranch, boardDir)` (`internal/fabricengine/clone.go:316-328`).
     On the seeding machine the board is then created on `master`, so `--force-with-lease=refs/lym/heads/main:` names a branch the board is not on.
     On a second machine, `ensureBoardWorktree` looks for `origin/master` in the records store, finds none, and orphan-creates an empty board.
   - Fix: D1 states that every reader of the board's branch as the primary's name, `primaryWeftBranch` included, takes the prime's code branch instead. D3 states that the clone creates or adopts the board on the fixed name. Alternatively, drop the fixed name and say how D3's probe finds the board.
2. **MEDIUM, design, `_lym/discussion/decision-record.md:117-119, 158-161`.** D3 replaces "a records branch an earlier failed attempt left", but it cannot tell such a leftover from a concurrent clone that is still in flight.
   - Counterexample: clone A has pushed `refs/lym/heads/&lt;p&gt;-weft` (tip T_A) and not yet its board. Clone B's precondition check observes T_A. B's records push is leased on T_A, so it lands T_B. A's board push, leased on absence, then lands.
   - Origin now holds A's board beside B's anchor-marker records branch until B's teardown restores T_A. If B is killed, or origin becomes unreachable before the restore, A's hub keeps a records branch on origin with B's anchor, possibly another `--subpath`, and A's local branch has diverged from it.
   - This breaks the Rationale's "a hub that reads as one on origin always carries its anchor" (line 166). The lease bounds only staleness, not ownership.
3. **LOW, design, `_lym/discussion/decision-record.md:132-134, 161`.** The bound "a clone whose board push fails leaves the records branch as it found it, unless another push moved it since" fails when origin becomes unreachable after the records push lands.
   - Counterexample: the board push fails on transport, the restore push fails the same way, and the hub directory is removed. Origin keeps the seed's records branch with no board.
   - A later `--single-repo` rerun replaces it, so the harm is small. The bound should name this case instead of claiming "as it found it".
4. **MEDIUM, design, `_lym/discussion/decision-record.md:143-153`.** "The check runs `git ls-remote` on origin once" cannot decide three of its own refusal arms: the default branch holding `.lym-warp`, or tracking a root `.lym-anchor` or `_lym`.
   - `ls-remote` lists refs, not trees. Today's file-content read is `probeWeftBinding`'s staged shallow clone (`internal/fabricengine/recordsprobe.go:60-77`), which also creates a probe directory in cwd, against D3's "before creating anything".
   - Fix: D3 names the probe that reads those paths, and its cost on a one-URL clone that takes the two-repository path.
5. **LOW, design, `_lym/discussion/decision-record.md:181-183`.** The `ls-remote`-then-delete check is not atomic.
   - Counterexample: a branch is present at the `ls-remote` and deleted by another machine's teardown before the push. The full-name push then exits 0 with `[deleted]`, and the destruction gate records a removal this call did not make.
   - Evidence: `internal/fabricengine/destroy.go:1631-1633` records on `deleted`. `remoteteardown.go:212` already treats "gone" as either deleter, so the impact is a mis-attributed mutation record.
6. **LOW, consistency, `_lym/discussion/decision-record.md:28, 181-182`.** D4 makes a two-repository hub's plain delete run `ls-remote` and push the full `refs/heads/` name. Today it pushes the short name and reads git's absent-ref error (`internal/gitrepo/push.go:313-323`).
   - That is a two-repository behaviour change: an extra network call and a new failure mode when `ls-remote` fails. It conflicts with Scope Out's "Any behaviour change to a two-repository hub beyond D7's reverts" and with D2's "the identity in a two-repository hub".
   - Fix: either keep the short-name path for the identity namespace, or list D4 in Scope Out's exception.

**Tool calls**

- `round-2-changes.md` and the `git log`/`git diff` stat of `_lym/discussion/`: to learn which hunks changed. The context lacked the changes list's content, and it names only the two files.
- `grep` for board-branch derivations outside `clone_onerepository.go`: to find a reader that ties the board's branch to the primary, one hop past `ensureBoardWorktree`.
- `cleanup.go:325-365` and `recordsBranchIn`: to confirm that `primaryWeftBranch` derives the prime's records branch from the board's branch. My context held only the grep hit.
